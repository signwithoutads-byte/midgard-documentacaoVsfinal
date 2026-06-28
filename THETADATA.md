# THETADATA.md — Integração ThetaData

## O que é o ThetaData

ThetaData é a fonte primária de dados de options do Midgard. Fornece:
- Options flow em tempo real (stream)
- Greeks (Delta, Gamma, Vega, Theta) por strike/expiration
- Snapshots EOD de OI e Greeks
- Dados históricos de options

**Site:** thetadata.net

---

## Como o Midgard usa o ThetaData

### Stream (tempo real)
```
ThetaData upstream
      │
      ▼
app/options_flow/stream.py — run_forever()
  └── conexão persistente, reconecta automaticamente
      │
      ▼
app/options_flow/enrichment.py
  └── enriquece com IV, SOFR, métricas
      │
      ▼
app/options_flow/state.py
  └── estado em memória da sessão
      │
      ▼
app/options_flow/broadcaster.py
  └── distribui via SSE para o frontend
```

### REST (sob demanda)
```
Frontend → GET /thetadata/[endpoint]
      │
      ▼
app/providers/thetadata/router.py
  └── proxy para ThetaData REST API
      │
      ▼
app/providers/thetadata/client.py
  └── cliente HTTP com auth
```

---

## Arquivos relevantes

| Arquivo | Função |
|---|---|
| `app/providers/thetadata/client.py` | Cliente HTTP, startup/shutdown, health check |
| `app/providers/thetadata/router.py` | Endpoints proxy REST |
| `app/options_flow/stream.py` | Stream upstream (principal — 11KB) |
| `app/options_flow/enrichment.py` | Enriquecimento de dados (6KB) |
| `app/options_flow/iv.py` | Cálculo de Implied Volatility |
| `app/options_flow/sofr.py` | Refresh loop da taxa SOFR |

---

## Autenticação

Feita via `THETADATA_API_KEY` nas variáveis de ambiente. O `client.py` injeta o header de autenticação em todas as requisições.

---

## Health check ThetaData

O endpoint `/health` do Midgard verifica dois aspectos:
- `thetadata_rest`: conectividade REST básica
- `thetadata_stream`: stream upstream ativo

Se o stream cair, o Midgard reconecta automaticamente via `run_forever()`.

---

## Pendências — o que falta implementar

### Hedge Flow
Os endpoints para o painel Hedge Flow ainda não existem no Midgard. Spec completo disponível em: `HEDGE_FLOW_BACKEND_SPEC_V1.4.md` (solicitar ao owner).

Dados necessários do ThetaData:
- Greeks EOD por strike (Delta, Gamma por expiration)
- OI por strike/expiration
- Snapshots intraday de Greeks

### Overnight Positioning
Spec: `OVERNIGHT_POSITIONING_BACKEND_SPEC_V1.0.md` (solicitar ao owner).

Rota planejada: `GET /api/overnight?symbol=QQQ&expiration_scope=ALL_45DTE`

Jobs necessários (não implementados):
- Job A: RTH Close preliminar ~15:59 ET
- Job B2: EOD Greeks principal ~17:15 ET
- Job C: RTH Close oficial via EOD ~17:15 ET
- Job D: sincronização OI polling 2min a partir de 06:25 ET

### Dark Pool
Dados virão do QuantData API (contrato pendente). Atualmente mock no frontend.

---

## Rate limits ThetaData

Uso atual estimado: ~20 req/min
Limite do plano: ~240 req/min
Margem disponível: ampla — irrelevante para número de subscribers.
