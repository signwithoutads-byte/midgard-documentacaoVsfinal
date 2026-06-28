# ONBOARDING_DEV.md — Guia para o Novo Desenvolvedor

## Contexto do projeto

**GEX Flow Options** é uma plataforma SaaS ($100/mês) de análise de options flow e GEX (Gamma Exposure) para traders intraday de futuros NQ/ES.

O produto tem dois componentes:
- **Frontend:** React/Vite/TypeScript hospedado via Lovable (`gexflowoptions.com.br`)
- **Backend (Midgard):** FastAPI/Python rodando em EC2 AWS via Docker/Coolify

Você vai trabalhar no **backend (Midgard)**.

---

## Leia primeiro (ordem obrigatória)

1. `ARCHITECTURE.md` — visão geral do sistema
2. `EC2.md` — acesso à infraestrutura
3. `COOLIFY.md` — deploy e gestão
4. `ENVIRONMENT.md` — variáveis de ambiente e credenciais
5. `THETADATA.md` — integração com a fonte de dados
6. `DEPLOY.md` — processo de deploy
7. `RECOVERY.md` — contexto de como o código foi obtido

---

## Setup local

### Pré-requisitos
- Python 3.11+
- Docker (para PostgreSQL local)
- Acesso ao ThetaData Terminal (solicitar API key ao owner)

### Passos

```bash
# Clone o repo
git clone [url-do-repo-midgard-api]
cd midgard-api

# Crie o ambiente virtual
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows

# Instale dependências
pip install -r requirements.txt

# Configure variáveis de ambiente
cp .env.example .env
# Edite .env com as credenciais fornecidas pelo owner

# Suba o banco local
docker run -d \
  --name midgard-pg \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:15

# Rode as migrations
alembic upgrade head

# Inicie o servidor
uvicorn app.main:app --reload --port 8000
```

### Verificar que está funcionando

```bash
curl http://localhost:8000/health
```

---

## O que está implementado

### ✅ Funcionando em produção

| Módulo | Localização | Descrição |
|---|---|---|
| Options Flow stream | `app/options_flow/stream.py` | Stream ThetaData em tempo real |
| Options Flow enrichment | `app/options_flow/enrichment.py` | IV, SOFR, métricas derivadas |
| Options Flow state | `app/options_flow/state.py` | Estado da sessão |
| Options Flow router | `app/options_flow/router.py` | Endpoints REST/SSE |
| Intraday | `app/intraday/router.py` | Dados intraday (Greeks, GEX) |
| ThetaData client | `app/providers/thetadata/client.py` | Cliente HTTP |
| Jobs EOD | `app/jobs/` | netdrift, iv_eod, vol_intraday, vol_eod |
| Auth | `app/auth.py` | Autenticação de requests |
| Cache | `app/cache/` | Cache de respostas |

---

## O que precisa ser implementado

### 🔴 Hedge Flow (prioridade 1)

Novo painel que mostra posicionamento de dealers baseado em Greek exposure.

**Spec completo:** solicitar `HEDGE_FLOW_BACKEND_SPEC_V1.4.md` ao owner.

Resumo técnico:
- Consome Greeks EOD + OI do ThetaData
- Calcula Dealer Delta, Gamma Exposure por strike
- Jobs de fechamento RTH: ~15:59 ET (preliminar) e ~17:15 ET (oficial)
- Rota: `GET /api/hedge-flow?symbol=QQQ`

### 🔴 Overnight Positioning (prioridade 2)

Painel pré-market (04:00–09:30 ET) que mostra posicionamento overnight dos dealers.

**Spec completo:** solicitar `OVERNIGHT_POSITIONING_BACKEND_SPEC_V1.0.md` ao owner.

Resumo técnico:
- Polling de OI a cada 2min a partir de 06:25 ET
- Calcula Dealer Delta @ Close e @ Projected Open
- Jobs A, B2, C, D (ver spec)
- Rota: `GET /api/overnight?symbol=QQQ&expiration_scope=ALL_45DTE`

### 🟡 Volatilidade + ML (prioridade 3)

Melhorias nos painéis de Volatilidade e ML Panel.

**Docs de handoff:** solicitar `DEV_VOLATILIDADE_MELHORIAS.md`, `DEV_SNAPSHOTS_VOL.md`, `DEV_ML_PANEL.md` ao owner.

Status: não confirmado se melhorias foram implementadas antes da transição.

### 🔴 Dark Pool (prioridade 4)

Atualmente 100% mock no frontend. Depende do QuantData API.

**Pendência:** contrato com QuantData não fechado em 27/06/2026. Confirmar com owner.

---

## Terminologia do produto

Termos que você vai encontrar nos specs e no código:

| Termo | Significado |
|---|---|
| GEX | Gamma Exposure — exposição gamma dos dealers |
| Call Wall / Put Wall | Nível de maior gamma de calls/puts (baseado em GEX, nunca Delta) |
| Gamma Flip | Ponto onde GEX muda de positivo para negativo |
| Flow State | Estado do options flow (bullish/bearish/neutral) |
| Flow Minute | Tab do painel Options Flow — dados por minuto |
| Flow Tape | Tab do painel Options Flow — tape de ordens |
| MAG7 Flow | Tab do painel Options Flow — Magnificent 7 |
| Net Premium | Premium líquido (bullish - bearish) |
| Dealer Delta | Delta acumulado dos dealers no mercado |
| OI | Open Interest |
| RTH | Regular Trading Hours (09:30–16:00 ET) |
| EOD | End of Day |

---

## Contato e recursos

- **Owner:** Paulo Padovani
- **Plataforma:** gexflowoptions.com.br
- **Coolify:** coolify.gexflowoptions.com.br
- **Specs de produto:** solicitar ao owner (congelados e documentados)
- **Frontend:** gerenciado via Lovable (owner tem acesso)

---

## Fluxo de trabalho esperado

1. Leia os specs antes de implementar qualquer endpoint
2. Implemente localmente e teste com `curl` ou Postman
3. Push para o repo GitHub
4. Deploy via Coolify (redeploy manual ou automático)
5. Verifique `/health` após deploy
6. Confirme com o owner que o painel no frontend está consumindo corretamente
