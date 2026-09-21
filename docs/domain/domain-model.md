# Modelo de domínio

## Princípio central

A **Avaliação** é o agregado central do EMIT.

Ela representa um trabalho técnico realizado sobre um aparelho/equipamento a pedido de um solicitante. O solicitante pode ser uma pessoa física ou uma empresa.

A avaliação concentra o estado atual do trabalho e referencia o histórico necessário para reconstruir como esse estado foi produzido.

## Modelo conceitual

```text
Installation
  |
  +-- Company
       +-- User
       |    +-- ADMINISTRATOR
       |    +-- TECHNICIAN
       |
       +-- Requester
       |     +-- Person
       |     +-- Organization
       |
       +-- Equipment
       |     +-- Category
       |     +-- Model
       |     +-- Component
       |
       +-- Evaluation
             +-- Requester
             +-- Equipment
             +-- Technician
             +-- TemplateVersion
             +-- Objective (texto opcional)
             +-- FieldResponse
             +-- TestResult
             +-- Attachment
             +-- EvaluationEvent
             +-- Report / Laudo
             |     +-- ReportVersion
             |           +-- Snapshot
             |           +-- PublicAccess
             |           +-- PDF
             |           +-- QR Code
             |
             +-- WorkOrderReference (opcional)
```

## Entidades

### Installation

Representa a configuração persistente de uma instalação do EMIT.

Uma instalação possui uma única configuração inicial e deve conhecer, no mínimo:

- status de inicialização;
- versão do sistema que criou/atualizou a configuração;
- empresa emissora dos laudos;
- domínio público canônico;
- preferências iniciais de templates.

A configuração de instalação não deve ser confundida com a configuração de ambiente do Docker.

### Company

Tenant da aplicação e empresa emissora dos laudos.

Responsável pelo isolamento dos dados de usuários, solicitantes, equipamentos, avaliações e configurações.

### User

Usuário autenticado.

Existem somente dois perfis:

- `ADMINISTRATOR`: configura a plataforma e suas estruturas;
- `TECHNICIAN`: executa avaliações e produz os registros técnicos.

O modelo deve evitar uma árvore complexa de permissões no MVP.

### Requester

Parte que solicita a avaliação.

Pode representar:

- pessoa física;
- empresa/organização.

O solicitante não é necessariamente um usuário da plataforma. Em regra, ele é apenas a parte relacionada ao atendimento/laudo.

### Equipment

Aparelho físico submetido à avaliação.

Possui categoria, modelo, identificadores e histórico de avaliações.

Um mesmo equipamento pode possuir várias avaliações ao longo de sua vida útil.

### Category

Categoria técnica do equipamento, como notebook, desktop, impressora, celular etc.

### Model

Modelo específico de um equipamento.

Pode herdar configurações da categoria e especializar campos/testes.

### Component

Componente ou periférico associado a um equipamento ou identificado durante uma avaliação.

### Evaluation

Objeto central do domínio.

Representa uma avaliação técnica realizada sobre um equipamento para um solicitante.

A avaliação possui:

- status;
- template/versionamento utilizado;
- técnico responsável;
- objetivo opcional em texto livre;
- respostas;
- testes;
- observações;
- evidências;
- conclusão;
- timestamps;
- histórico de alterações;
- laudos emitidos.

O objetivo é **semântico**, não uma entidade estruturada. O sistema não precisa manter uma lista fechada de objetivos. O técnico pode escrever, por exemplo, "estado de entrada", "estado de saída", "diagnóstico para orçamento" ou outro contexto adequado.

### Template

Estrutura configurável para orientar uma avaliação.

É composto por seções, campos e testes.

### TemplateVersion

Versão imutável da estrutura de um template utilizada por uma avaliação.

Uma alteração no template não deve modificar retroativamente a estrutura histórica de avaliações já iniciadas.

### Section

Agrupa campos e testes relacionados.

### Field

Campo configurável de uma seção.

### FieldResponse

Resposta dada pelo técnico a um campo durante uma avaliação.

O armazenamento deve suportar valores flexíveis, preservando o tipo do campo e permitindo auditoria.

### Test

Definição reutilizável de teste técnico.

Pode possuir instruções, resultados possíveis, obrigatoriedade e regras para evidências.

### TestResult

Resultado de um teste dentro de uma avaliação.

Deve registrar resultado, observação, responsável e momento da execução.

### Attachment

Arquivo de evidência relacionado à avaliação, campo, seção ou teste.

O conteúdo binário fica no storage S3-compatible; o banco mantém metadados e referência ao objeto.

### EvaluationEvent

Evento imutável da linha do tempo de uma avaliação.

Exemplos:

- avaliação criada;
- avaliação iniciada;
- campo preenchido;
- campo alterado;
- teste executado;
- evidência adicionada/removida;
- observação alterada;
- status alterado;
- avaliação concluída;
- laudo emitido;
- avaliação reaberta;
- alteração após emissão;
- novo laudo emitido;
- acesso público revogado.

O evento deve registrar, no mínimo:

- avaliação;
- usuário responsável;
- data/hora;
- tipo do evento;
- entidade/campo afetado, quando aplicável;
- valor anterior e/ou novo valor quando necessário;
- metadados adicionais.

Eventos são append-only no fluxo normal da aplicação.

### Report / Laudo

Representação final e publicável de uma avaliação.

O laudo não deve ser entendido apenas como uma tela derivada do estado atual da avaliação. Uma emissão cria um registro próprio que preserva a representação documental daquele momento.

O laudo deve apresentar, conforme aplicável:

- empresa emissora;
- solicitante;
- equipamento;
- objetivo da avaliação;
- respostas, testes, observações e conclusão;
- técnico responsável;
- local;
- data e hora de emissão;
- local para assinatura;
- identificador do documento;
- QR Code e endereço de consulta.

Uma avaliação pode existir sem laudo, especialmente enquanto estiver em rascunho ou em andamento.

### ReportVersion

Cada emissão do laudo corresponde a uma versão.

Exemplo:

```text
Evaluation #123
   |
   +-- ReportVersion 1
   |      +-- snapshot do conteúdo emitido
   |      +-- PDF v1
   |      +-- PublicAccess v1
   |
   +-- ReportVersion 2
          +-- snapshot após reabertura
          +-- PDF v2
          +-- PublicAccess v2
```

Uma versão emitida é **imutável**.

A versão deve preservar o snapshot dos dados utilizados na emissão, incluindo a representação necessária para reconstruir o documento sem depender do estado atual da avaliação.

### Snapshot

Representação congelada dos dados relevantes da avaliação no momento da emissão do laudo.

O snapshot deve ser suficiente para que uma versão histórica continue apresentando o conteúdo que foi efetivamente emitido, mesmo que a avaliação seja posteriormente reaberta e alterada.

O formato exato do snapshot será definido na implementação. Ele não deve ser confundido com um dump do banco.

### PublicAccess

Controle de publicação de uma **versão específica do laudo**.

Possui, no mínimo:

- token público aleatório e não enumerável;
- status;
- possibilidade de expiração/revogação;
- regras de visibilidade;
- referência à versão do laudo publicada.

A URL pública deve apontar para a versão do documento, não para o estado mutável atual da avaliação.

O QR Code deve carregar somente a URL/token de consulta. Não deve carregar os dados do laudo diretamente.

### PDF

O PDF é uma representação derivada da versão imutável do laudo.

Se for regenerado tecnicamente, deve continuar sendo derivado do mesmo snapshot. O conteúdo documental não pode acompanhar alterações posteriores da avaliação.

### WorkOrderReference

Referência opcional a uma ordem de serviço.

Não é entidade central do domínio. Serve para conectar a avaliação a um sistema ou processo operacional externo.

Pode armazenar:

- código da OS;
- URL;
- identificador externo;
- sistema de origem.

## Estado da avaliação

Estados iniciais sugeridos:

```text
DRAFT
  |
  v
IN_PROGRESS
  |
  v
COMPLETED
  |
  +--> REPORT_ISSUED
  |
  +--> REOPENED
           |
           v
       IN_PROGRESS
```

`CANCELLED` pode existir como estado terminal conforme as regras de negócio.

A emissão do laudo não deve apagar ou substituir o estado técnico da avaliação. Ela cria um marco documental e de integridade.

## Regra de bloqueio após emissão

Após a emissão de qualquer laudo para uma avaliação:

1. a avaliação entra em estado protegido contra edição normal;
2. ações de edição comuns devem ser recusadas pela camada de aplicação;
3. uma alteração exige uma operação explícita de reabertura;
4. a reabertura deve exigir autenticação/autorização compatível com o perfil e a política definida;
5. a reabertura gera um `EvaluationEvent`;
6. toda alteração posterior relevante gera novos eventos;
7. o laudo anteriormente emitido permanece imutável;
8. uma nova conclusão pode gerar uma nova versão do laudo.

A autenticação deve ser baseada no usuário do EMIT. Não é necessário criar uma "senha da avaliação" no modelo de domínio.

## Regra de versionamento do laudo

```text
Avaliação
   |
   +-- emissão --> Laudo v1 [imutável]
   |
   +-- reabertura --> alterações auditadas
                         |
                         +-- nova conclusão
                                |
                                +-- emissão --> Laudo v2 [imutável]
```

Cada versão possui seu próprio momento de emissão e snapshot.

O QR Code de v1 deve continuar consultando v1 mesmo depois da emissão de v2.

A existência de v2 não altera nem apaga v1. A aplicação pode indicar que existe uma versão posterior, conforme as regras de exposição pública, mas o conteúdo de v1 permanece preservado.

## Preenchimento flexível

A estrutura configurada orienta o técnico, mas não deve transformar a avaliação em um formulário rígido.

Regras:

1. campos podem ser opcionais ou obrigatórios conforme template;
2. avaliações podem ser salvas parcialmente;
3. o técnico pode retornar à avaliação enquanto ela estiver editável;
4. observações livres podem complementar campos estruturados;
5. o objetivo pode contextualizar a avaliação sem criar uma taxonomia obrigatória;
6. evidências podem ser adicionadas durante o processo;
7. alterações relevantes geram eventos;
8. depois de um laudo emitido, alterações exigem reabertura;
9. o estado atual não substitui o histórico;
10. uma versão de laudo nunca deve depender do estado atual para reconstruir seu conteúdo.

## Timeline x estado atual

São conceitos diferentes:

- **Estado atual**: permite abrir a avaliação e continuar o trabalho rapidamente.
- **Timeline**: permite entender o que aconteceu, quando aconteceu e quem fez cada alteração.
- **Laudo**: representa o documento emitido em um momento específico.

A timeline não deve ser derivada apenas comparando registros atuais. Eventos de alteração devem ser persistidos.

## Regra de versionamento de templates

Templates são configuráveis pelo Administrador, mas a avaliação deve apontar para uma versão específica.

Assim:

```text
Template
  |
  +-- Version 1 ----> Avaliações antigas
  |
  +-- Version 2 ----> Novas avaliações
```

Isso impede que uma alteração administrativa mude retroativamente o formulário utilizado em uma avaliação.

## Regra sobre OS

A OS é uma integração/referência operacional opcional:

```text
Solicitante -> Avaliação <- Equipamento
                    |
                    +---- Técnico
                    |
                    +---- OS (opcional)
```

Essa decisão mantém o domínio focado no trabalho técnico e permite integrar posteriormente com sistemas externos.

## InitialSetup

Representa o processo de primeira execução.

Deve garantir que somente uma configuração inicial seja concluída para a instalação. O fluxo inclui:

1. dados da empresa;
2. endereço e telefone;
3. usuário Administrador inicial;
4. usuário Técnico opcional;
5. escolha entre templates iniciais ou configuração limpa;
6. domínio público inicial.

O processo deve ser transacional e idempotente.

## ExportPackage

Não é necessário persistir o arquivo exportado como entidade de domínio permanente.

O arquivo deve possuir um envelope versionado contendo, conceitualmente:

- `format`: identificação do formato;
- `formatVersion`: versão do formato;
- `product`: EMIT;
- `systemVersion`: versão do EMIT que gerou o arquivo;
- `exportedAt`: data/hora da exportação;
- `templates`: templates e versões exportadas;
- `metadata`: informações auxiliares necessárias à importação.

Exemplo conceitual:

```json
{
  "format": "emit-template-package",
  "formatVersion": 1,
  "product": "EMIT",
  "systemVersion": "0.1.0",
  "exportedAt": "2026-09-21T00:00:00Z",
  "templates": []
}
```

O formato deve ser tratado como contrato de interoperabilidade, não como dump do PostgreSQL.

### Compatibilidade de importação

A importação deve validar em etapas:

1. formato reconhecido;
2. versão do formato suportada;
3. versão do EMIT de origem;
4. recursos utilizados pelo pacote;
5. conflitos com templates existentes.

Se houver migração possível, o sistema deve indicar a migração antes de concluir a importação.

## PublicInstallationConfig

Configuração pública persistente da instalação.

Inclui o domínio canônico, por exemplo:

```text
https://emit.dominio.com.br
```

Esse valor é usado para:

- navegação e links absolutos;
- URLs públicas de versões de laudos;
- QR Codes;
- metadados;
- referências geradas pela aplicação.

A variável de ambiente pode servir apenas como valor inicial/default de bootstrap.

## Relação com Company

A empresa cadastrada na primeira execução é a entidade emissora dos laudos. Ela não deve ser tratada apenas como um valor técnico de configuração.

```text
Installation
   |
   +-- Company (emissora)
   +-- PublicInstallationConfig
   +-- Users
   +-- Templates
   +-- Evaluations
```

## Primeira execução

```text
Deploy
  |
  v
Banco vazio / instalação não inicializada
  |
  v
Initial Setup
  |
  +--> Empresa
  +--> Endereço / telefone
  +--> Administrador
  +--> Técnico (opcional)
  +--> Templates iniciais ou configuração limpa
  +--> Domínio público
  |
  v
Instalação inicializada
  |
  v
Aplicação normal
```
