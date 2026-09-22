# Arquitetura

## Princípios

1. Monólito modular inicialmente.
2. A Avaliação é o agregado central do domínio.
3. Separação entre apresentação, aplicação, domínio e infraestrutura.
4. PostgreSQL como fonte de verdade dos dados estruturados.
5. Arquivos grandes fora do banco, em storage S3-compatible.
6. Estruturas de avaliação dirigidas por configuração e versionamento.
7. Preenchimento progressivo com salvamento de estado parcial.
8. O estado da avaliação deve permanecer simples, limitado a `DRAFT`, `IN_PROGRESS`, `COMPLETED` e `CANCELLED`.
9. Histórico de alterações persistente e append-only.
10. Timeline da avaliação baseada em eventos, não somente em comparação de estados.
11. Após emissão, um laudo é imutável.
12. Alterações posteriores exigem reabertura explícita e são auditadas.
13. Versões de laudo preservam snapshots da composição documental emitida.
14. Identificadores públicos não enumeráveis.
15. Preparação para jobs assíncronos sem microsserviços prematuros.
16. Deploy reproduzível via Docker Compose.

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
- reports
- public-access
- qrcode
- audit

### Módulos secundários

- work-order-reference
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
                   +--> Objetivo (texto opcional)
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
                   +--> emissão
                          |
                          v
                       Laudo
                          |
                          +--> ReportVersion
                          |      +--> Snapshot
                          |      +--> PDF
                          |      +--> PublicAccess
                          |      +--> QR Code
                          |
                          +--> nova versão após reabertura
```

## Objetivo da avaliação

O objetivo é um campo textual opcional da avaliação.

Ele existe para contextualizar o trabalho para quem preenche, revisa ou lê o laudo. Não deve ser transformado em entidade ou enum obrigatório.

Exemplos:

- estado de entrada;
- estado de saída;
- diagnóstico;
- inspeção;
- avaliação para orçamento;
- manutenção;
- outro contexto descrito pelo técnico.

A ausência do objetivo não impede a avaliação.

## Histórico, bloqueio e consistência

Toda operação relevante sobre uma avaliação deve atualizar o estado atual e, na mesma transação, registrar o evento correspondente.

Exemplo conceitual:

```text
BEGIN
UPDATE evaluation
INSERT evaluation_event
COMMIT
```

Quando a operação for uma emissão de laudo, a transação deve estabelecer o marco de integridade da avaliação e registrar o evento de emissão.

Após a emissão de um laudo:

1. a avaliação mantém seu estado técnico e recebe um marco de proteção contra edição normal;
2. o bloqueio deve ser tratado separadamente do status da avaliação;
3. uma operação explícita de reabertura deve ser executada antes de qualquer alteração;
3. a reabertura deve ser autorizada pelo mecanismo de acesso do EMIT;
4. reabertura e alterações posteriores geram eventos;
5. a avaliação pode ser concluída novamente;
6. uma nova emissão gera uma nova versão do laudo.

Não é necessário criar uma senha exclusiva para cada avaliação. A identidade do usuário autenticado e a auditoria da operação são os mecanismos de rastreabilidade.

Eventos não devem ser editados ou apagados pelo fluxo normal da aplicação.

## Laudos e snapshots
### Evidências e mídia

O módulo `attachments` representa as evidências técnicas associadas às avaliações.

No MVP, devem ser suportadas pelo menos três categorias de mídia:

- `IMAGE`;
- `AUDIO`;
- `VIDEO`.

O arquivo binário deve permanecer no storage S3-compatible/MinIO e seus metadados no PostgreSQL.

A interface deve fornecer reprodução/visualização básica:

| Mídia | Apresentação MVP |
|---|---|
| IMAGE | visualização |
| AUDIO | controles nativos de reprodução |
| VIDEO | controles nativos de reprodução |

A implementação inicial pode utilizar os recursos nativos do navegador. Para captura de áudio/vídeo, a implementação pode utilizar APIs de mídia do navegador, como `getUserMedia` e `MediaRecorder`. A gravação deve ser iniciada a partir da própria tela da avaliação e, após confirmação, o resultado deve ser vinculado automaticamente à avaliação. Se a permissão de câmera/microfone for negada ou o recurso não estiver disponível, a interface deve informar o problema sem bloquear o restante da avaliação. Não é necessário criar um player multimídia proprietário nem implementar transcodificação ou streaming adaptativo no MVP.

A evidência pode ser vinculada à avaliação, seção, campo ou teste. O técnico decide quais evidências participam da composição do laudo. Evidências não selecionadas continuam sendo materiais internos da avaliação, sujeitas às permissões de acesso.

Ao emitir uma `ReportVersion`, qualquer evidência incorporada ao documento deve ser congelada por referência de armazenamento imutável.


O módulo `reports` representa a emissão documental da avaliação.

Uma emissão cria uma versão imutável:

```text
Evaluation
   |
   +-- ReportVersion 1
   |      +-- Snapshot
   |      +-- PDF
   |      +-- PublicAccess
   |
   +-- reopen + changes
   |
   +-- ReportVersion 2
          +-- Snapshot
          +-- PDF
          +-- PublicAccess
```

O snapshot é a representação congelada da composição documental utilizada para gerar aquela versão. O Laudo é uma representação documental da Avaliação: combina informações registradas pelo técnico e outras informações relevantes e as organiza conforme o template e os padrões de apresentação do EMIT. O Snapshot preserva os dados e a composição necessários para reproduzir o documento, e não é um dump do banco.

O Snapshot pode preservar, conforme aplicável, emissor, solicitante, equipamento, objetivo, local, técnico, campos e resultados, testes, observações, conclusão, evidências selecionadas, textos/rótulos, template/versão, identidade visual e metadados de emissão. Evidências que integram o documento devem ter referências de armazenamento imutáveis.

A mesma ReportVersion pode ser apresentada por consulta digital pública ou como PDF destinado à impressão A4. A consulta pública é uma representação do laudo e não deve expor automaticamente todos os dados existentes na Avaliação ou no Snapshot; regras de publicação devem minimizar dados pessoais e sensíveis.

O laudo não deve ser renderizado novamente a partir do estado atual da avaliação para consultas históricas. A consulta de uma versão deve utilizar seu snapshot.

O QR Code deve apontar para a publicação daquela versão. A existência de uma versão posterior não modifica a versão anterior.

O PDF é uma representação derivada da versão emitida. O espaço para assinatura, técnico responsável, local, data e hora fazem parte da composição documental do laudo.

No MVP, "assinatura" representa o espaço/registro previsto no documento. Não implica assinatura digital com certificado.

## Publicação e validação

A publicação pública deve ser associada a uma versão específica do laudo.

```text
ReportVersion
      |
      +--> PublicAccess
      |      +--> random token
      |      +--> visibility rules
      |      +--> status / revocation
      |
      +--> QR Code
      |
      +--> public URL
```

O token não deve ser enumerável nem conter o conteúdo do documento.

A consulta pública deve apresentar a versão publicada do laudo e permitir verificar sua situação de publicação.

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
- links públicos de versões de laudos;
- QR Codes;
- canonical URLs;
- Open Graph/metadados;
- links enviados por notificações futuras.

A variável `PUBLIC_BASE_URL` pode ser mantida para bootstrap, fallback de instalação ainda não configurada ou ambientes de desenvolvimento, mas não deve ser a única fonte da URL pública depois que a instalação estiver configurada.

A aplicação deve validar o domínio configurado e não deve confiar cegamente no cabeçalho HTTP `Host` para construir URLs.
