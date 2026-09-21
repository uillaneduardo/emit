# Modelo de domínio

## Princípio central

A **Avaliação** é o agregado central do EMIT.

Ela representa um trabalho técnico realizado sobre um aparelho/equipamento a pedido de um solicitante. O solicitante pode ser uma pessoa física ou uma empresa.

A avaliação concentra o estado atual do trabalho e referencia o histórico necessário para reconstruir como esse estado foi produzido.

## Modelo conceitual

```text
Company
  +-- User
       +-- ADMINISTRATOR
       +-- TECHNICIAN
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
        +-- FieldResponse
        +-- TestResult
        +-- Attachment
        +-- EvaluationEvent
        +-- PublicAccess
        +-- WorkOrderReference (opcional)
```

## Entidades

### Company

Tenant da aplicação.

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
- tipo;
- técnico responsável;
- template/versionamento utilizado;
- respostas;
- testes;
- observações;
- evidências;
- conclusão;
- timestamps;
- histórico de alterações;
- acesso público, quando publicado.

Uma avaliação deve poder existir sem OS.

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
- laudo publicado;
- acesso público revogado.

O evento deve registrar, no mínimo:

- avaliação;
- usuário responsável;
- data/hora;
- tipo do evento;
- entidade/campo afetado, quando aplicável;
- valor anterior e/ou novo valor quando necessário;
- metadados adicionais.

### PublicAccess

Controle de publicação de uma avaliação/laudo.

Possui token público aleatório, status de publicação, possibilidade de expiração/revogação e regras de visibilidade.

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
  +--> REVIEWED / REOPENED (se necessário)
  |
  v
CANCELLED
```

O fluxo exato deve ser definido antes da implementação do domínio.

## Preenchimento flexível

A estrutura configurada orienta o técnico, mas não deve transformar a avaliação em um formulário rígido.

Regras:

1. campos podem ser opcionais ou obrigatórios conforme template;
2. avaliações podem ser salvas parcialmente;
3. o técnico pode retornar à avaliação;
4. observações livres podem complementar campos estruturados;
5. evidências podem ser adicionadas durante o processo;
6. alterações relevantes geram eventos;
7. o estado atual não substitui o histórico.

## Timeline x estado atual

São conceitos diferentes:

- **Estado atual**: permite abrir a avaliação e continuar o trabalho rapidamente.
- **Timeline**: permite entender o que aconteceu, quando aconteceu e quem fez cada alteração.

A timeline não deve ser derivada apenas comparando registros atuais. Eventos de alteração devem ser persistidos.

## Regra de versionamento

Templates são configuráveis pelo Administrador, mas a avaliação deve apontar para uma versão específica.

Assim:

```text
Template
  |
  +-- Version 1 ----> Avaliações antigas
  |
  +-- Version 2 ----> Novas avaliações
```

Isso impede que uma alteração administrativa mude retroativamente o formulário utilizado em um laudo já produzido.

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
