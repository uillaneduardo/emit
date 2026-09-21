# EMIT

Plataforma PWA técnica para emissão, preenchimento, controle e consulta de laudos de equipamentos de TI.

## Conceito central

A **Avaliação** é o objeto central da plataforma.

Uma avaliação:

- é realizada sobre um aparelho/equipamento;
- é solicitada por uma pessoa física ou por uma empresa;
- é executada por um técnico;
- pode permanecer em rascunho e ser preenchida progressivamente;
- reúne dados estruturados, observações, testes e evidências;
- mantém histórico de alterações;
- possui uma linha do tempo consultável;
- pode originar um laudo para consulta interna ou pública.

A plataforma não será limitada a um checklist rígido. O administrador configura a estrutura dos tipos de avaliação, mas o preenchimento deve permitir registrar a realidade encontrada pelo técnico sem perder rastreabilidade.

## Perfis

A plataforma terá no máximo dois perfis:

- **Administrador**: configura a plataforma, usuários, categorias, modelos, campos, tipos de avaliação e regras de preenchimento.
- **Técnico**: realiza avaliações, registra resultados, adiciona evidências, acompanha avaliações e emite/atualiza laudos conforme as permissões definidas.

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
                +--> Estrutura/Template
                +--> Respostas e testes
                +--> Evidências
                +--> Histórico
                +--> Timeline
                +--> Laudo
                +--> Consulta pública / QR Code
```

Uma OS pode existir como referência operacional, mas não é o centro do domínio. Ela pode ser vinculada à avaliação quando houver integração ou necessidade de controle externo.

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
