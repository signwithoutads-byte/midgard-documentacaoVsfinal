# RECOVERY.md — Contexto de Recuperação do Código

## O que aconteceu

Em junho de 2026, houve uma transição de desenvolvedor. O código fonte do Midgard estava no repositório do desenvolvedor anterior e não foi entregue formalmente.

O código foi recuperado diretamente da instância EC2 de produção via backup do servidor (`/data/coolify/applications/`), representando o estado exato do código que estava rodando em produção em **27/06/2026**.

---

## O que foi recuperado

O backup contém o código funcional completo do Midgard:

```
app/
├── cache/
├── intraday/router.py          ✅ completo
├── jobs/                       ✅ 4 jobs implementados
├── models/
├── options_flow/               ✅ completo (stream, enrichment, state, iv, sofr)
├── providers/thetadata/        ✅ client + router implementados
├── auth.py                     ✅
├── config.py                   ✅
├── db.py                       ✅
└── main.py                     ✅
alembic/                        ✅ migrations existentes
scripts/
entrypoint.sh                   ✅
```

---

## O que pode estar incompleto

Com base nos specs de produto existentes, os seguintes módulos ainda não estavam implementados:

- **Hedge Flow** — nenhum arquivo `hedge_flow/` encontrado no backup
- **Overnight Positioning** — nenhum arquivo `overnight/` encontrado no backup
- **Dark Pool** — dependente do QuantData API (contrato não fechado em 27/06/2026)

---

## O que não foi recuperado

- Histórico de commits e branches do Git
- PRs, issues e comentários de código
- Variáveis de ambiente em texto plano (ficam no Coolify, não no código)
- Configurações internas do Coolify além do que está no `docker-compose`

---

## Credenciais rotacionadas após a transição

| Serviço | Status |
|---|---|
| SSH EC2 (key pair) | ✅ Rotacionado 27/06/2026 |
| AWS IAM | ✅ Usuário anterior deletado |
| Coolify | ✅ Usuário anterior removido |
| GitHub | ✅ Colaborador removido |
| Lovable | ✅ Colaborador removido |
| ThetaData API key | ⚠️ Verificar com owner |
| Supabase | Gerenciado pela Lovable |

---

## Próximos passos recomendados

1. Criar repositório GitHub próprio para o `midgard-api`
2. Fazer commit do código recuperado como commit inicial
3. Vincular o novo repo ao Coolify (substituir o repo anterior)
4. Implementar os módulos pendentes (Hedge Flow, Overnight Positioning)
5. Rotacionar ThetaData API key após setup do novo dev
