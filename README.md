# EMIT

Plataforma PWA técnica para emissão, preenchimento, controle e consulta de laudos de equipamentos de TI.

## Conceito central

A **Avaliação** é o objeto central da plataforma.

Uma avaliação:

- é realizada sobre um aparelho/equipamento;
- é solicitada por uma pessoa física ou por uma empresa;
- é executada por um técnico;
- pode permanecer em rascunho e ser preenchida progressivamente;
- possui um **objetivo opcional**, em texto livre, que contextualiza o motivo da avaliação;
- reúne dados estruturados, observações, testes e evidências;
- mantém histórico de alterações;
- possui uma linha do tempo consultável;
- possui estados simples de trabalho (`DRAFT`, `IN_PROGRESS`, `COMPLETED` ou `CANCELLED`);
- pode originar um ou mais laudos.

O objetivo da avaliação não é uma classificação estrutural obrigatória. Ele é um texto semântico preenchido pelo técnico, podendo registrar contextos como estado de entrada, estado de saída, diagnóstico, inspeção, manutenção ou orçamento.

A plataforma não será limitada a um checklist rígido. O administrador configura a estrutura dos tipos de avaliação, mas o preenchimento deve permitir registrar a realidade encontrada pelo técnico sem perder rastreabilidade.

## Perfis

A plataforma terá no máximo dois perfis:

- **Administrador**: configura a plataforma, usuários, categorias, modelos, campos, tipos de avaliação e regras de preenchimento.
- **Técnico**: realiza avaliações, registra resultados, adiciona evidências, acompanha avaliações e emite laudos conforme as permissões definidas.

O controle de acesso deve ser simples e orientado ao uso técnico. Não haverá uma hierarquia extensa de perfis no MVP.

## Domínio principal

```text
Solicitante (Pessoa Física ou Empresa)
                |
                v
             Avaliação
                |
                +--> Equipamento
                +--> Técnico responsável
                +--> TemplateVersion
                +--> Objetivo (texto opcional)
                +--> Respostas e testes
                +--> Evidências
                +--> Histórico / Timeline
                |
                +--> Laudo
                       +--> versão imutável
                       +--> Snapshot
                       +--> PDF / A4
                       +--> consulta pública digital
                       +--> QR Code
```

Uma OS pode existir como referência operacional, mas não é o centro do domínio. Ela pode ser vinculada à avaliação quando houver integração ou necessidade de controle externo.

## Integridade após emissão do laudo

A avaliação permanece como registro técnico, mas a emissão de um laudo cria um marco de integridade.

Depois da emissão:

- a avaliação fica bloqueada para edição normal;
- uma alteração posterior exige **reabertura explícita** e autorização;
- a reabertura exige um **motivo obrigatório**, independentemente do perfil autorizado;
- a reabertura e as alterações ficam registradas na auditoria/timeline;
- o laudo já emitido nunca é alterado;
- uma nova emissão gera uma nova versão do laudo;
- cada versão do laudo preserva um snapshot imutável da composição documental utilizada na emissão.

A instalação define se a reabertura pode ser executada por **Somente Administrador** ou por **Administrador e Técnico**. Essa configuração pode ser alterada posteriormente nas configurações administrativas.

Assim, o QR Code e a consulta pública de uma versão continuam representando exatamente o documento emitido naquele momento.

## Laudo

O **Laudo** é uma representação documental da Avaliação, composta a partir das informações registradas pelo técnico e de outras informações relevantes, organizada conforme o template e os padrões de apresentação do EMIT.

O laudo não é uma simples cópia dos dados da avaliação. Ele organiza e apresenta essas informações de forma amigável, padronizada e adequada ao meio de entrega.

A mesma versão emitida pode ser apresentada em diferentes meios:

- consulta digital por URL pública;
- consulta através de QR Code;
- PDF destinado à impressão em formato A4.

O laudo deve preservar o conteúdo documental apresentado no momento da emissão, independentemente de alterações posteriores na avaliação, nos cadastros, no template ou na identidade visual atual do sistema.

A versão pública não deve expor automaticamente todos os dados existentes na avaliação. A apresentação pública deve aplicar as regras de publicação e minimização de informações pessoais e sensíveis.

O documento deve apresentar, conforme o contexto:

- empresa emissora;
- solicitante, conforme o que o laudo determinar como relevante;
- equipamento;
- objetivo da avaliação, quando informado;
- conteúdo e resultados da avaliação;
- testes;
- observações;
- conclusão;
- nome do técnico responsável;
- local;
- data e hora de emissão;
- local para assinatura;
- identificador do laudo;
- QR Code e endereço para consulta.

O QR Code deve apontar para um identificador/token público não enumerável, e não carregar o conteúdo do laudo diretamente.

## Snapshot do laudo
### Evidências e arquivos

A avaliação deve aceitar evidências em pelo menos três mídias no MVP:

- imagem;
- áudio;
- vídeo.

A evidência é um registro técnico associado à avaliação e, quando aplicável, a uma seção, campo ou teste. O arquivo binário fica no storage S3-compatible/MinIO, enquanto os metadados ficam no PostgreSQL.

O MVP deve oferecer reprodução/visualização básica diretamente na aplicação:

- imagens devem possuir visualização;
- áudios devem possuir controles nativos de reprodução;
- vídeos devem possuir controles nativos de reprodução;
- arquivos incompatíveis com reprodução direta devem continuar disponíveis para consulta/download conforme o tipo e as permissões.

Além do upload, o técnico poderá iniciar uma gravação de áudio ou vídeo diretamente na tela da avaliação. Após encerrar a gravação, poderá confirmar ou cancelar o material; ao confirmar, a mídia será armazenada e vinculada automaticamente à avaliação e ao contexto em que a gravação foi iniciada.

A reprodução não precisa de um player avançado no MVP. O objetivo é permitir que o técnico visualize ou reproduza o conteúdo sem depender obrigatoriamente de uma ferramenta externa. A captura dependerá das permissões e capacidades do navegador; falhas de permissão ou suporte não devem impedir o restante da avaliação.

Nem toda evidência precisa fazer parte do laudo. A inclusão no laudo é uma decisão de composição documental. Uma evidência pode permanecer apenas como material interno da avaliação.

Quando uma evidência for utilizada em uma versão emitida do laudo, o conteúdo utilizado deve ser preservado de forma imutável para que a versão histórica não dependa do arquivo atual da avaliação.


Cada `ReportVersion` deve possuir um **Snapshot imutável** que represente a composição documental daquela versão.

O Snapshot deve preservar os dados e informações necessários para reproduzir o conteúdo do laudo sem depender do estado atual mutável da avaliação.

O Snapshot não é um dump do banco de dados. Ele representa o documento emitido e deve incluir, conforme aplicável:

- dados da empresa emissora apresentados no documento;
- solicitante apresentado no documento;
- identificação do equipamento;
- objetivo;
- local;
- técnico responsável;
- campos e resultados;
- testes;
- observações;
- conclusão;
- evidências selecionadas para o laudo;
- textos e rótulos relevantes para a apresentação;
- referência ao template/versão utilizado;
- identidade visual necessária à composição;
- data/hora de emissão;
- identificação da versão;
- informações necessárias à consulta e validação.

Evidências que fazem parte do laudo devem possuir referências de armazenamento imutáveis, de forma que substituição ou exclusão posterior do arquivo original não altere uma versão já emitida.

A existência de uma informação no Snapshot não significa que ela deva ser exibida publicamente. A camada de publicação determina a representação pública do laudo.

## Arquitetura

Monólito modular, preparado para self-hosting com Docker Compose.

```text
Internet -> Cloudflare -> Reverse Proxy/Tunnel -> EMIT -> PostgreSQL + MinIO
```

## Stack planejada

- Frontend: Next.js + TypeScript
- Backend: NestJS + TypeScript
- Banco: PostgreSQL + Prisma
- Storage: MinIO/S3
- Editor: Tiptap
- UI: Tailwind CSS + shadcn/ui
- Deploy: Docker Compose

## Estrutura

```text
apps/
  web/
  api/
packages/
  ui/
  config/
  types/
prisma/
docs/
docker/
docker-compose.yml
.env.example
```

## Configuração

Copie `.env.example` para `.env` e preencha os valores reais. O `.env` não é versionado. O Docker Compose utiliza interpolação de variáveis do ambiente.

## Documentação

- docs/architecture/overview.md
- docs/domain/domain-model.md
- docs/requirements/requirements.md
- docs/deployment/self-hosted.md
- docs/deployment/environment.md
- docs/decisions/ADR-001-evaluation-report-integrity.md
