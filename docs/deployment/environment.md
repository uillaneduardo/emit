# Variáveis de ambiente

.env.example é a referência versionada das configurações necessárias.

Em produção:

```text
.env -> docker compose -> containers
```

O docker-compose.yml deve usar interpolação, por exemplo:

```yaml
POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

Nunca inserir credenciais reais diretamente no YAML ou no código.

- .env.example: versionado.
- .env: somente no ambiente local/servidor.
- Segredos reais: nunca no Git.
