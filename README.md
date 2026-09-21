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
                       +--> PDF
                       +--> consulta pública
                       +--> QR Code
```

Uma OS pode existir como referência operacional, mas não é o centro do domínio. Ela pode ser vinculada à avaliação quando houver integração ou necessidade de controle externo.

## Integridade após emissão do laudo

A avaliação permanece como registro técnico, mas a emissão de um laudo cria um marco de integridade.

Depois da emissão:

- a avaliação fica bloqueada para edição normal;
- uma alteração posterior exige **reabertura explícita** e autorização;
- a reabertura e as alterações ficam registradas na auditoria/timeline;
- o laudo já emitido nunca é alterado;
- uma nova emissão gera uma nova versão do laudo;
- cada versão do laudo preserva um snapshot do conteúdo utilizado na emissão.

Assim, o QR Code e a consulta pública de uma versão continuam representando exatamente o documento emitido naquele momento.

## Laudo

O **Laudo** é a representação final e publicável de uma avaliação.

Ele pode ser:

- gerado em PDF;
- consultado por uma URL pública;
- validado por QR Code;
- impresso ou compartilhado.

O documento deve apresentar, conforme o contexto:

- empresa emissora;
- solicitante;
- equipamento;
- objetivo da avaliação, quando informado;
- conteúdo e resultados da avaliação;
- conclusão;
- nome do técnico responsável;
- local;
- data e hora de emissão;
- local para assinatura;
- identificador do laudo;
- QR Code e endereço para consulta.

O QR Code deve apontar para um identificador/token público não enumerável, e não carregar o conteúdo do laudo diretamente.

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
