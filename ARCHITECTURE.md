# ARCHITECTURE.md — Arquitetura do Sistema GEX Flow Options

## Visão geral

```
[Usuário/Trader]
      │
      ▼
[Frontend — React/Vite/TypeScript]
  Hospedado via Lovable
  gexflowoptions.com.br
      │
      │ HTTPS (REST + SSE)
      ▼
[Midgard — FastAPI/Python]
  EC2 AWS us-east-1
  Docker via Coolify
  35.172.139.19
      │
      ├──── PostgreSQL (banco interno, opcional)
      │
      └──── ThetaData Terminal
              (dados de options em tempo real)
```

---

## Componentes

### Frontend
- **Stack:** React + Vite + TypeScript
- **Plataforma:** Lovable (editor visual + deploy)
- **Auth:** Supabase (gerenciado pela Lovable)
- **Domínio:** gexflowoptions.com.br

### Midgard (este repo)
- **Stack:** Python + FastAPI
- **Função:** proxy centralizado de dados de mercado
- **Deploy:** Docker container gerenciado pelo Coolify
- **Instância:** EC2 `i-04a475c61e51f2251`

### ThetaData Terminal
- Fonte primária de dados: options flow, Greeks, snapshot EOD
- Conexão via REST e stream (WebSocket/SSE interno)
- Autenticação via API key nas variáveis de ambiente

### PostgreSQL
- Banco interno do Midgard
- Armazena snapshots intraday e EOD calculados pelos jobs
- Opcional: sistema faz graceful degrade sem banco

### Coolify
- Painel de deploy self-hosted rodando na mesma EC2
- URL: coolify.gexflowoptions.com.br
- Gerencia o container Docker do Midgard

---

## Fluxo de dados — Options Flow (principal)

```
ThetaData Stream
      │
      ▼
stream.py — run_forever()
  └── recebe eventos brutos do upstream
      │
      ▼
enrichment.py
  └── enriquece com IV, SOFR, métricas derivadas
      │
      ▼
state.py
  └── mantém estado da sessão em memória
      │
      ▼
broadcaster.py
  └── distribui para clientes conectados via SSE
      │
      ▼
router.py — GET /options-flow/stream
  └── endpoint SSE consumido pelo frontend
```

---

## Fluxo de dados — Jobs EOD

```
APScheduler (cron)
      │
      ├── netdrift_cron (1min)    → calcula net drift
      ├── iv_eod_cron (16:30 ET) → snapshot IV
      ├── vol_intraday_cron (1min)→ vol intraday
      └── vol_eod_cron (16:45 ET)→ snapshot vol EOD
            │
            ▼
        PostgreSQL
```

---

## Painéis do frontend e seus endpoints no Midgard

| Painel | Status | Endpoint Midgard |
|---|---|---|
| Strike Map | ✅ Produção | `/thetadata/*` |
| Options Flow | ✅ Produção | `/options-flow/*` |
| Greeks Profile | ✅ Produção | `/intraday/*` |
| Volatilidade | ⚠️ Parcial | `/intraday/*` |
| Cockpits QQQ/SPY | ✅ Produção | `/intraday/*` |
| ML Panel | ⚠️ Incompleto | — |
| Day Levels | ✅ Produção | `/intraday/*` |
| Dark Pool | 🔴 Mock | pendente QuantData |
| Hedge Flow | 🔴 Pendente | spec: HEDGE_FLOW_BACKEND_SPEC |
| Overnight Positioning | 🔴 Pendente | spec: OVERNIGHT_POSITIONING_BACKEND_SPEC |
| IA Analyzer | 🔴 Pendente | — |

---

## Segurança

- SSH: key pair `gexflow_nova.pem` (rotacionado em 27/06/2026)
- Porta 22: restrita ao IP do owner
- Porta 8000/6001/6002: restrita ao IP do Coolify
- Portas 80/443: abertas (tráfego público)
