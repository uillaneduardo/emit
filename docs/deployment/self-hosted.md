# Deploy self-hosted

## Pré-requisitos

- Linux
- Docker Engine
- Docker Compose Plugin
- DNS no Cloudflare
- armazenamento persistente

## Configuração

```bash
git clone https://github.com/uillaneduardo/emit.git
cd emit
cp .env.example .env
nano .env
```

Preencha as credenciais reais no .env. Nunca faça commit desse arquivo.

## Subida

```bash
docker compose build
docker compose up -d
```

## Verificação

```bash
docker compose ps
docker compose logs -f
```

## Persistência

Os volumes postgres_data e minio_data são críticos e não devem ser removidos durante atualizações.

## Cloudflare

O acesso público ficará atrás do Cloudflare. O desenho final poderá usar Cloudflare Tunnel ou reverse proxy local, conforme a infraestrutura disponível.
