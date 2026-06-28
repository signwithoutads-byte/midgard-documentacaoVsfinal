# EC2.md — Acesso e Gestão da Instância EC2

## Dados da instância

| Campo | Valor |
|---|---|
| Instance ID | `i-04a475c61e51f2251` |
| Nome | Gex Flow Options Data Feed |
| IP Público | `35.172.139.19` |
| IP Privado | `172.31.41.130` |
| Região | us-east-1 (N. Virginia) |
| OS | Ubuntu 24.04 LTS |
| Key pair ativo | `gexflow_nova.pem` (rotacionado 27/06/2026) |

---

## Acesso SSH

```bash
ssh -i /caminho/para/gexflow_nova.pem ubuntu@35.172.139.19
```

**Usuário:** `ubuntu` (não `root`)

A chave `.pem` é mantida pelo owner do projeto. Devs recebem acesso via Coolify, não via SSH direto.

---

## Security Group (`launch-wizard-1`)

| Porta | Protocolo | Source | Descrição |
|---|---|---|---|
| 22 | TCP | IP do owner /32 | SSH — restrito |
| 80 | TCP | 0.0.0.0/0 | HTTP público |
| 443 | TCP | 0.0.0.0/0 | HTTPS público |
| 6001 | TCP | 177.137.237.245/32 | Coolify Websocket |
| 6002 | TCP | 177.137.237.245/32 | Coolify Terminal |
| 8000 | TCP | 177.137.237.245/32 | Coolify UI |

**Importante:** Para usar EC2 Instance Connect via console AWS, é necessário adicionar temporariamente o range `18.206.0.0/15` na porta 22 e remover após uso.

---

## Estrutura de diretórios relevante

```
/data/coolify/          # Dados do Coolify (apps, configs, volumes)
/data/coolify/applications/[id]/   # Container do Midgard
```

Acesso como `ubuntu` requer `sudo` para entrar em `/data/coolify/`.

---

## Volumes Docker (Midgard)

| Volume | Descrição |
|---|---|
| `theta-data` | Dados persistentes ThetaData |
| `redis-data` | Cache Redis |
| `midgard-pgdata` | Dados PostgreSQL |

---

## Operações comuns

**Ver containers rodando:**
```bash
sudo docker ps
```

**Ver logs do Midgard:**
```bash
sudo docker logs [container_id] --tail 100 -f
```

**Reiniciar Midgard:**
Preferível via Coolify (painel) em vez de docker diretamente.

**Health check:**
```bash
curl https://[dominio]/health
```
