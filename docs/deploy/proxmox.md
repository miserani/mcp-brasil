# Deploy no Proxmox (ingest-hub)

Este fork roda numa VM do Proxmox, dentro do cluster k3s do [ingest-hub](https://github.com/miserani/ingest-hub). O grafo CNPJ chama o `mcp-brasil` nas seções "Ao vivo" (Compras.gov, Portal da Transparência, TCU, Querido Diário, TSE, Transferegov).

## A esteira

| Workflow | Quando | O que faz |
|---|---|---|
| `sync-upstream.yml` | todo dia às 06:17 (Brasília), ou à mão | Traz o `main` de [Mcp-Brasil/mcp-brasil](https://github.com/Mcp-Brasil/mcp-brasil) para o `main` deste fork. Um conflito para a esteira e deixa a mudança para um merge manual. |
| `CI` (do projeto original) | a cada push no `main` | Lint, tipos e testes. |
| `deploy-proxmox.yml` | depois de um CI verde no `main`, ou à mão | Roda **na própria VM** (runner self-hosted com o rótulo `mcp-brasil-proxmox`). O `/usr/local/sbin/mcp-brasil-deploy` da VM põe o clone no commit, builda `mcp-brasil:latest`, reinicia o Deployment `mcp-brasil` e confere o `/health`. Nunca roda em pull request. |

**O token do sync:**

- **Com o secret `SYNC_TOKEN`:** um token com *Contents* e *Workflows* de escrita neste fork. O merge do sync é um push normal: o CI roda e, verde, o deploy segue.
- **Sem ele:** o sync usa o `GITHUB_TOKEN`, cujos pushes não disparam workflows. O sync então chama o deploy direto, sem passar pelo CI. Uma mudança do projeto original em `.github/workflows` falha até o `SYNC_TOKEN` existir.

## As chaves

As chaves ficam na VM, não no GitHub. No repositório do ingest-hub:

```bash
cp cloud/proxmox/secrets.env.example cloud/proxmox/secrets.env   # preencha a seção [mcp-brasil]
cloud/proxmox/vm.sh secrets
```

O grafo usa a `TRANSPARENCIA_API_KEY` (Contratos com o governo federal). As outras fontes que ele consulta não pedem chave.

## Na VM

```bash
cloud/proxmox/vm.sh mcp-brasil   # instala o mcp-brasil-deploy e registra o runner (uma vez)
cloud/proxmox/vm.sh ssh 'sudo cat /var/lib/ingest-hub/state/mcp-brasil-commit'   # commit no ar
```
