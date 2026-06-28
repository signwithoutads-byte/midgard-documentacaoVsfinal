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

## O que está funcionando em produção — revisar antes de começar

Antes de implementar qualquer coisa nova, valide que esses painéis estão operando corretamente:

| Painel | Observação |
|---|---|
| Greeks Profile | Funcionando |
| Strike Map | Funcionando |
| Day Levels | Funcionando |
| Flow Tape | Tab dentro de Options Flow |
| MAG7 Flow | Tab dentro de Options Flow |

QQQ Cockpit e SPY Cockpit se conectam automaticamente — não requerem implementação adicional.

---

## Ordem de prioridades — o que implementar

### 1. Flow Minute + Minute Bar (dentro de Options Flow)
Tab já existente no frontend mas com implementação pendente no backend.
Minute Bar = 1 barra por minuto de NET FLOW acumulado.
Solicitar spec ao owner.

### 2. Hedge Flow
Novo painel de posicionamento de dealers baseado em Greek exposure.
Spec completo disponível: solicitar `HEDGE_FLOW_BACKEND_SPEC_V1.4.md` ao owner.

Resumo técnico:
- Consome Greeks EOD + OI do ThetaData
- Calcula Dealer Delta, Gamma Exposure por strike
- Jobs de fechamento: ~15:59 ET (preliminar) e ~17:15 ET (oficial)
- Rota: `GET /api/hedge-flow?symbol=QQQ`

### 3. Overnight Positioning
Painel pré-market (04:00–09:30 ET) de posicionamento overnight dos dealers.
Spec completo disponível: solicitar `OVERNIGHT_POSITIONING_BACKEND_SPEC_V1.0.md` ao owner.

Resumo técnico:
- Polling de OI a cada 2min a partir de 06:25 ET
- Jobs A, B2, C, D (ver spec)
- Rota: `GET /api/overnight?symbol=QQQ&expiration_scope=ALL_45DTE`

### 4. Volatilidade
Melhorias no painel existente.
Docs de handoff: solicitar `DEV_VOLATILIDADE_MELHORIAS.md` e `DEV_SNAPSHOTS_VOL.md` ao owner.

### 5. ML Panel
Melhorias no painel existente.
Doc de handoff: solicitar `DEV_ML_PANEL.md` ao owner.

### 6. Market Setup
Novo painel. Solicitar spec ao owner.

### 7. Dark Pool
Atualmente 100% mock no frontend.
Depende de contrato com QuantData API (pendente com owner).
Não implementar backend até contrato fechado — confirmar com owner antes de iniciar.

---

## Terminologia do produto

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
- **Specs de produto:** solicitar ao owner (todos documentados e congelados)
- **Frontend:** gerenciado via Lovable (owner tem acesso)

---

## Fluxo de trabalho esperado

1. Leia os specs antes de implementar qualquer endpoint
2. Implemente localmente e teste com `curl` ou Postman
3. Push para o repo `midgard-api` no GitHub
4. Deploy via Coolify (redeploy manual)
5. Verifique `/health` após deploy
6. Confirme com o owner que o painel no frontend está consumindo corretamente
