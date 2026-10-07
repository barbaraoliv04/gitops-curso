# Prática 3 — Sync automático, drift e ambientes

**Aula 3 · 14/10 · 45 minutos**

Objetivo: criar os ambientes dev e prod com Kustomize, deixar o Argo CD sincronizar sozinho,
ver o self-heal em ação e desfazer uma mudança ruim com `git revert`.

## Passo 1 — Limpar a aula anterior

```bash
kubectl delete -f argocd/aula2/web.yaml
kubectl get all -n curso
```

O `finalizer` da Application faz o Argo CD apagar também o Deployment, o Service e o ConfigMap.

## Passo 2 — Entender os overlays

Compare o que cada ambiente gera, sem aplicar nada:

```bash
kubectl kustomize apps/web/overlays/dev  | grep -E "namespace|replicas|image"
kubectl kustomize apps/web/overlays/prod | grep -E "namespace|replicas|image"
```

Anote o que muda entre dev e prod e em quais arquivos isso está definido.

## Passo 3 — Criar os dois ambientes

```bash
cat argocd/apps/web-dev.yaml     # repare no bloco syncPolicy.automated
kubectl apply -f argocd/apps/web-dev.yaml
kubectl apply -f argocd/apps/web-prod.yaml
```

Sem clicar em SYNC, as duas apps devem ficar **Synced** e **Healthy**.

```bash
kubectl get pods -n web-dev
kubectl get pods -n web-prod
```

## Passo 4 — Self-heal

```bash
kubectl scale deploy/web -n web-prod --replicas=10
kubectl get pods -n web-prod -w      # Ctrl+C para sair
```

Depois tente apagar o Service de dev e observe:

```bash
kubectl delete svc web -n web-dev
kubectl get svc -n web-dev
```

## Passo 5 — Promover uma versão

1. Edite `apps/web/overlays/dev/index.html` → `Versao: v2`. Commit e push.
2. Confira em http://localhost:8082 (`kubectl port-forward svc/web -n web-dev 8082:80`).
3. Prod continua em v1? Confira em http://localhost:8083 (`kubectl port-forward svc/web -n web-prod 8083:80`).
4. Agora promova: faça a mesma mudança em `apps/web/overlays/prod/index.html`, commit e push.

## Passo 6 — Prune

1. Crie o arquivo `apps/web/overlays/dev/aviso.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aviso
data:
  mensagem: "Manutenção sexta às 22h"
```

2. Adicione `- aviso.yaml` na lista `resources` de `apps/web/overlays/dev/kustomization.yaml`.
3. Commit e push. Confira: `kubectl get configmap aviso -n web-dev`.
4. Remova o arquivo e a linha do `kustomization.yaml`, commit e push. O ConfigMap continua no cluster?

## Passo 7 — Um commit ruim e o rollback

1. Em `apps/web/overlays/dev/kustomization.yaml`, adicione no final:

```yaml
images:
  - name: nginx
    newTag: versao-que-nao-existe
```

2. Commit (`"Atualiza nginx em dev"`) e push. Observe a app **web-dev** na interface:
   clique no Pod novo e veja **EVENTS**.
3. Desfaça pelo Git:

```bash
git log --oneline -3
git revert HEAD --no-edit
git push
```

4. Confirme que **web-dev** voltou a **Healthy**.

## Evidência de conclusão

- **web-dev** e **web-prod** Synced e Healthy, as duas mostrando **Versao: v2**.
- `git log --oneline` mostrando o commit ruim e o commit de revert.
- ConfigMap `aviso` criado e depois removido pelo Argo CD.

## Perguntas de checagem

1. O que `selfHeal: true` fez no passo 4? E o que teria acontecido sem ele?
2. Qual opção do `syncPolicy` removeu o ConfigMap `aviso`?
3. Por que o rollback foi feito com `git revert` e não pelo botão ROLLBACK da interface?
4. Que arquivo você mudaria para prod ter 5 réplicas?
