# EMIT

Plataforma PWA para checklist, avaliação técnica e laudos digitais de equipamentos de TI.

## Objetivo

Registrar digitalmente entrada, identificação, diagnóstico, testes, evidências, avaliação técnica e saída de equipamentos, com consulta pública por link e QR Code.

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

Copie .env.example para .env e preencha os valores reais. O .env não é versionado. O Docker Compose utiliza interpolação de variáveis do ambiente.

## Documentação

- docs/architecture/overview.md
- docs/domain/domain-model.md
- docs/requirements/requirements.md
- docs/deployment/self-hosted.md
- docs/deployment/environment.md
