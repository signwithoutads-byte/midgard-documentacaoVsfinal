# COOLIFY.md — Gestão via Coolify

## O que é o Coolify

Coolify é o painel de deploy self-hosted que gerencia o container Docker do Midgard. Roda na mesma EC2 do Midgard.

**URL:** `coolify.gexflowoptions.com.br`

---

## Acesso

- Login via magic link (email)
- Owner: `paulopadovani@icloud.com`
- Time ativo: **Root Team** (onde ficam os recursos)

> Atenção: existe também o `paulopadovani's Team` que está vazio. Os projetos ficam no **Root Team**.

---

## Estrutura no Coolify

```
Root Team
├── Projects
│   └── Midgard          ← backend proxy
│       └── [ambiente]
│           ├── Environment Variables
│           ├── Deployments
│           └── Logs
└── Servers
    └── localhost        ← a própria EC2
```

---

## Operações via painel

**Deploy novo código:**
1. Root Team → Projects → Midgard → [ambiente]
2. Botão **Deploy** ou **Redeploy**
3. Coolify faz pull do repositório e reconstrói o container

**Ver variáveis de ambiente:**
1. Root Team → Projects → Midgard → [ambiente]
2. Aba **Environment Variables**

**Ver logs em tempo real:**
1. Root Team → Projects → Midgard → [ambiente]
2. Aba **Logs**

**Terminal no container:**
1. Root Team → Servers → localhost
2. Aba **Terminal**

---

## Portas expostas pelo Coolify

| Porta | Uso |
|---|---|
| 6001 | Websocket do painel |
| 6002 | Terminal do painel |
| 8000 | Interface web |

Todas restritas ao IP do Coolify cloud (`177.137.237.245/32`) no Security Group da EC2.

---

## Adicionando novo dev

1. Root Team → Teams → Members → Invite New Member
2. Inserir email do dev
3. Role recomendado: **Member** (não owner)

> Transactional Emails precisa estar configurado para convite por email funcionar. Alternativa: gerar invitation link manualmente.

---

## Repositório vinculado

O Coolify faz deploy a partir de um repositório GitHub. Para verificar qual repo está vinculado:

```bash
sudo docker inspect coolify | grep REPOSITORY
```

Após troca de dev, verificar se o repo vinculado é o do owner do projeto, não do dev anterior.
