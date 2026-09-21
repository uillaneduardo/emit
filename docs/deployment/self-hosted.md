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

Preencha as credenciais reais no `.env`. Nunca faça commit desse arquivo.

O `PUBLIC_BASE_URL` pode ser informado no deploy como valor inicial/default. Após a inicialização, o domínio funcional da instalação é persistido no banco e passa a ser a fonte de verdade para URLs públicas, laudos e QR Codes.

## Subida

```bash
docker compose build
docker compose up -d
```

## Primeira execução

Após subir uma instalação nova:

1. o EMIT detecta que a instalação ainda não foi inicializada;
2. apresenta o assistente de configuração inicial;
3. cadastra a empresa emissora, endereço e telefone;
4. cria o primeiro Administrador;
5. permite criar um Técnico opcional;
6. permite escolher templates iniciais ou configuração limpa;
7. configura/confirma o domínio público.

A aplicação somente libera o uso normal após a conclusão consistente do bootstrap.

## Verificação

```bash
docker compose ps
docker compose logs -f
```

## Persistência

Os volumes `postgres_data` e `minio_data` são críticos e não devem ser removidos durante atualizações.

Os dados de avaliações, histórico, laudos, versões de laudos e configurações de instalação devem ser incluídos na estratégia de backup do PostgreSQL. Os arquivos associados aos laudos/evidências devem ser incluídos no backup do storage S3-compatible.

## Cloudflare

O acesso público ficará atrás do Cloudflare. O desenho final poderá usar Cloudflare Tunnel ou reverse proxy local, conforme a infraestrutura disponível.

O serviço de banco e o storage não devem ser expostos diretamente à Internet.
