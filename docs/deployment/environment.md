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

## Domínio público

A variável `PUBLIC_BASE_URL` pode ser utilizada como configuração inicial/default durante o deploy, por exemplo:

```env
PUBLIC_BASE_URL=https://emit.dominio.com.br
```

Porém, ela **não é a fonte definitiva do domínio funcional da instalação**.

Durante a primeira execução, o domínio público deve ser confirmado/configurado e persistido no banco como parte da configuração da instalação. Depois disso, a aplicação deve utilizar a configuração persistida para gerar URLs públicas, links absolutos, QR Codes e metadados.

Isso permite alterar o domínio pela configuração da própria instalação sem exigir que todas as referências funcionais dependam exclusivamente do `.env`.

O `.env` continua sendo responsável por configurações de infraestrutura e segredos, não por substituir as configurações funcionais persistidas da instalação.
