# Arquitetura

## Princípios

1. Monólito modular inicialmente.
2. A Avaliação é o agregado central do domínio.
3. Separação entre apresentação, aplicação, domínio e infraestrutura.
4. PostgreSQL como fonte de verdade dos dados estruturados.
5. Arquivos grandes fora do banco, em storage S3-compatible.
6. Estruturas de avaliação dirigidas por configuração e versionamento.
7. Preenchimento progressivo com salvamento de estado parcial.
8. Histórico de alterações persistente e append-only.
9. Timeline da avaliação baseada em eventos, não somente em comparação de estados.
10. Identificadores públicos não enumeráveis.
11. Preparação para jobs assíncronos sem microsserviços prematuros.
12. Deploy reproduzível via Docker Compose.

## Perfis

O sistema deve manter apenas dois perfis no domínio:

- `ADMINISTRATOR`: configuração da plataforma;
- `TECHNICIAN`: operação técnica e preenchimento das avaliações.

A autorização deve ser simples. Não criar uma hierarquia de papéis genéricos sem necessidade.

## Módulos previstos

- auth
- companies
- users
- requesters
- equipment
- categories
- models
- templates
- evaluations
- tests
- attachments
- public-access
- qrcode
- audit

### Módulos secundários

- work-order-reference
- reports
- notifications

A referência de OS não deve dominar o desenho do domínio.

## Fluxo arquitetural principal

```text
Administrador
    |
    +--> configura categorias/modelos
    +--> configura templates e testes
    +--> configura regras de publicação
                         |
                         v
                    TemplateVersion
                         |
                         v
Solicitante --> Avaliação <-- Equipamento
                   |
                   +--> Técnico
                   +--> Respostas
                   +--> Testes
                   +--> Evidências
                   +--> Estado atual
                   |
                   +--> EvaluationEvent
                   |       |
                   |       v
                   |    Timeline
                   |
                   +--> Laudo
                           |
                           +--> Consulta pública / QR Code
```

## Histórico e consistência

Toda operação relevante sobre uma avaliação deve atualizar o estado atual e, na mesma transação, registrar o evento correspondente.

Exemplo conceitual:

```text
UPDATE evaluation
INSERT evaluation_event
COMMIT
```

Se uma alteração de resposta ocorrer, o evento deve identificar o campo afetado e, quando necessário, preservar o valor anterior e o novo valor.

Eventos não devem ser editados ou apagados pelo fluxo normal da aplicação.

## Versionamento de templates

O template editável é uma configuração administrativa.

Quando uma versão for utilizada por uma avaliação, sua estrutura deve ser imutável para aquela avaliação.

```text
Template
  |
  +-- TemplateVersion 1
  |       +-- Evaluation A
  |       +-- Evaluation B
  |
  +-- TemplateVersion 2
          +-- Evaluation C
```

## Self-hosting

Produção: Docker Engine, Docker Compose, volumes persistentes para PostgreSQL e MinIO e publicação através do Cloudflare. O `.env` pertence ao ambiente de implantação e não ao repositório.

PostgreSQL e MinIO não devem ser publicados diretamente na Internet. O acesso externo deve ocorrer somente pelos serviços necessários da aplicação/reverse proxy.


## Bootstrap e configuração inicial

A instalação possui um estado persistente de inicialização. Na primeira execução, quando ainda não houver configuração concluída, a aplicação deve direcionar o usuário para o assistente de configuração inicial.

O assistente configura:

1. empresa emissora;
2. endereço e telefone;
3. Administrador inicial;
4. Técnico inicial opcional;
5. templates iniciais ou configuração limpa;
6. domínio público canônico.

O bootstrap deve ser executado de forma transacional. Após concluído, as rotas normais da aplicação ficam disponíveis.

A configuração persistida é distinta do `.env`: o ambiente fornece infraestrutura e valores de bootstrap; o banco guarda configurações funcionais da instalação.

## Exportação/importação de templates

Templates devem possuir um formato de intercâmbio próprio do EMIT. Não utilizar dump de tabelas ou dados específicos do Prisma/PostgreSQL como formato de exportação.

O pacote deve conter um envelope com pelo menos:

```text
format
formatVersion
product
systemVersion
exportedAt
templates
metadata
```

A importação deve verificar o `formatVersion` antes do conteúdo e depois avaliar a compatibilidade com a versão do sistema indicada em `systemVersion`.

A compatibilidade deve ser baseada em regras explícitas de versão. Quando necessário, uma versão futura poderá implementar migradores:

```text
package v1 -> importer v2 -> migration -> internal model
```

O processo de importação deve ser precedido por validação/dry-run e confirmação do Administrador. Conflitos não devem ser resolvidos por sobrescrita silenciosa.

## Domínio público da instalação

O domínio público da instalação é uma configuração funcional persistida no banco.

Exemplo:

```text
https://emit.dominio.com.br
```

Ele deve ser utilizado por toda a aplicação na geração de URLs absolutas, incluindo:

- links de navegação que necessitem de URL absoluta;
- links públicos de avaliações/laudos;
- QR Codes;
- canonical URLs;
- Open Graph/metadados;
- links enviados por notificações futuras.

A variável `PUBLIC_BASE_URL` pode ser mantida para bootstrap, fallback de instalação ainda não configurada ou ambientes de desenvolvimento, mas não deve ser a única fonte da URL pública depois que a instalação estiver configurada.

A aplicação deve validar o domínio configurado e não deve confiar cegamente no cabeçalho HTTP `Host` para construir URLs.
