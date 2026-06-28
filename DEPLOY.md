# DEPLOY.md — Processo de Deploy

## Stack de deploy

```
GitHub (repo midgard-api)
      │
      ▼
Coolify (pull automático ou manual)
      │
      ▼
Docker build (usando entrypoint.sh)
      │
      ▼
Container rodando na EC2
      │
      ▼
Nginx/proxy reverso (porta 443)
      │
      ▼
Endpoint público acessível pelo frontend
```

---

## Deploy via Coolify (padrão)

1. Acessa `coolify.gexflowoptions.com.br`
2. Root Team → Projects → Midgard → [ambiente]
3. Clica **Redeploy**
4. Coolify faz pull do branch configurado, rebuild e restart do container
5. Acompanha em tempo real na aba **Deployments**

---

## entrypoint.sh

Script de inicialização do container. Executado pelo Docker na subida.

Localização: raiz do repo `midgard-api/entrypoint.sh`

Responsabilidades típicas:
- Rodar migrations Alembic (`alembic upgrade head`)
- Iniciar o servidor Uvicorn/Gunicorn com FastAPI

---

## Migrations de banco (Alembic)

```bash
# Criar nova migration
alembic revision --autogenerate -m "descrição"

# Aplicar migrations
alembic upgrade head

# Ver histórico
alembic history
```

As migrations ficam em `alembic/versions/`.

Em produção, o `entrypoint.sh` roda `alembic upgrade head` automaticamente na subida do container.

---

## Variáveis de ambiente no deploy

Todas injetadas pelo Coolify — não existe `.env` em disco.

Ver `ENVIRONMENT.md` para lista completa.

---

## Rollback

Via Coolify:
1. Root Team → Projects → Midgard → Deployments
2. Seleciona deployment anterior
3. Clica **Redeploy this deployment**

---

## Verificar deploy bem-sucedido

```bash
curl https://[dominio]/health
```

Resposta esperada:
```json
{
  "status": "ok",
  "providers": {
    "thetadata_rest": "healthy",
    "thetadata_stream": "connected",
    "postgres": "healthy"
  }
}
```

Se `status: degraded`, verificar logs no Coolify.
