# Arquitetura

## Princípios

1. Monólito modular inicialmente.
2. Separação entre apresentação, aplicação, domínio e infraestrutura.
3. PostgreSQL como fonte de verdade dos dados estruturados.
4. Arquivos grandes fora do banco, em storage S3-compatible.
5. Checklists dirigidos por configuração.
6. Identificadores públicos não enumeráveis.
7. Preparação para jobs assíncronos sem microsserviços prematuros.
8. Deploy reproduzível via Docker Compose.

## Módulos previstos

- auth
- companies
- users
- permissions
- customers
- equipment
- categories
- models
- work-orders
- templates
- evaluations
- tests
- components
- attachments
- public-access
- qrcode
- audit

## Self-hosting

Produção: Docker Engine, Docker Compose, volumes persistentes para PostgreSQL e MinIO e publicação através do Cloudflare. O .env pertence ao ambiente de implantação e não ao repositório.
