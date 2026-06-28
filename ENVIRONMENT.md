# ENVIRONMENT.md — Variáveis de Ambiente

## ⚠️ Aviso

Todas as credenciais foram rotacionadas em 27/06/2026. Antes de qualquer deploy, obtenha os valores atuais com o owner do projeto.

**Nunca commite valores reais de credenciais no repositório.**

---

## Variáveis obrigatórias

| Variável | Descrição | Onde obter |
|---|---|---|
| `THETADATA_API_KEY` | API key do ThetaData Terminal | painel ThetaData |
| `DATABASE_URL` | Connection string PostgreSQL | Coolify → Midgard → Environment |
| `CORS_ORIGINS` | Origins permitidas (frontend URL) | configuração do projeto |

## Variáveis opcionais

| Variável | Descrição | Default |
|---|---|---|
| `LOG_LEVEL` | Nível de log (`INFO`, `DEBUG`) | `INFO` |
| `CORS_ORIGIN_REGEX` | Regex de origins permitidas | — |

---

## Onde as variáveis ficam em produção

As variáveis de ambiente são injetadas pelo **Coolify** no container Docker do Midgard:

```
Coolify → Root Team → Projects → Midgard → [ambiente] → Environment Variables
```

Não existe arquivo `.env` em disco — tudo é gerenciado pelo Coolify.

---

## Comportamento sem DATABASE_URL

O sistema faz **graceful degrade**:
- ThetaData stream funciona normalmente
- Options flow funciona normalmente
- Jobs APScheduler **não iniciam** (netdrift, iv_eod, vol_intraday, vol_eod)
- Health check retorna `postgres: disabled`

---

## Rotação de credenciais

Ao onboarding de novo dev:
1. **Não compartilhe** a API key do ThetaData diretamente
2. Acesse o Coolify e adicione o novo dev como membro do Root Team
3. As variáveis ficam visíveis dentro do Coolify para membros autorizados
4. Se precisar rotacionar: ThetaData → Account → API Keys → Regenerate
