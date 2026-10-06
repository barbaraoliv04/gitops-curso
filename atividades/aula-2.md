# Prática 2 — Argo CD e a primeira Application

**Aula 2 · 08/10 · 45 minutos**

Objetivo: instalar o Argo CD, conectar o seu fork e fazer o primeiro deploy a partir de um commit.

## Passo 1 — Instalar o Argo CD

Com o Docker Desktop aberto e o Kubernetes verde:

```bash
kubectl config use-context docker-desktop
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.5.3/manifests/install.yaml
```

Acompanhe os Pods subirem (pode levar alguns minutos). Espere todos ficarem `Running` e `1/1`:

```bash
kubectl get pods -n argocd -w      # Ctrl+C para sair
```

Pegue a senha inicial do usuário `admin`. Ela fica no Secret `argocd-initial-admin-secret`,
no campo `password`, codificada em base64:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

Guarde a senha.

Em um terminal separado, deixe rodando:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse https://localhost:8080, aceite o aviso de certificado e entre com `admin` e a senha.

## Passo 2 — Apontar para o seu fork

As Applications da pasta `argocd/` ainda apontam para um repositório de exemplo:

```yaml
repoURL: https://github.com/SEU_USUARIO/gitops-curso.git
```

1. Abra **cada arquivo** abaixo no editor e troque `SEU_USUARIO` pelo seu usuário do GitHub:
   - `argocd/aula2/web.yaml`
   - `argocd/apps/web-dev.yaml`
   - `argocd/apps/web-prod.yaml`
   - `argocd/root.yaml`
2. Confira se não sobrou nenhum:

```bash
grep -rn "SEU_USUARIO" argocd/      # não deve mostrar nada
```

3. Envie:

```bash
git add argocd
git commit -m "Aponta Applications para o meu fork"
git push
```

## Passo extra (opcional) — Verificar o Git com mais frequência

Por padrão o Argo CD olha o Git a cada 2 a 3 minutos. Para a aula, dá para reduzir para ~1 minuto
mudando o ConfigMap `argocd-cm`:

```bash
kubectl -n argocd patch configmap argocd-cm --type merge \
  -p '{"data":{"timeout.reconciliation":"60s"}}'
kubectl -n argocd rollout restart statefulset argocd-application-controller
kubectl -n argocd rollout restart deployment argocd-repo-server
```

Se pular este passo, use o botão **REFRESH** sempre que quiser que o Argo CD veja um commit novo.

## Passo 3 — Criar a Application

Leia o arquivo antes de aplicar:

```bash
cat argocd/aula2/web.yaml
kubectl apply -f argocd/aula2/web.yaml
```

Na interface, abra o cartão **web**:

- Qual é o status de **Sync**? E de **Health**?
- Clique em **APP DIFF**. O que está diferente?
- Clique em **SYNC → SYNCHRONIZE**.

## Passo 4 — Deploy por commit

1. Em `apps/web/base/deployment.yaml`, mude `replicas: 2` para `replicas: 3`.
2. Commit e push:

```bash
git commit -am "Escala web para 3 réplicas"
git push
```

3. Na interface, clique em **REFRESH** (ou espere a verificação automática). A app deve ficar **OutOfSync**.
4. Veja o **APP DIFF** e depois clique em **SYNC**.
5. Confirme no terminal:

```bash
kubectl get pods -n curso
```

## Passo 5 — Mudar a página

1. Em `apps/web/base/index.html`, troque para `Versao: v3`.
2. Commit, push, refresh e sync.
3. Na árvore de recursos, observe o **novo ConfigMap** (com outro sufixo) e o **novo ReplicaSet**.
4. Confira em http://localhost:8081 (refaça o `port-forward svc/web -n curso 8081:80` se tiver caído).

## Passo 6 — Mudança fora do Git

```bash
kubectl scale deploy/web -n curso --replicas=1
```

Atualize a interface. O que o Argo CD mostra? Clique em **SYNC** e veja o resultado.

## Evidência de conclusão

- Application **web** em **Synced** e **Healthy**, com 3 Pods.
- Em **HISTORY AND ROLLBACK**, pelo menos dois syncs, cada um com o SHA do commit.
- Página mostrando **Versão: v3**.

## Perguntas de checagem

1. No passo 3, por que a Application começou **OutOfSync** mesmo com os recursos já existindo?
2. Qual a diferença entre **Refresh** e **Sync**?
3. No passo 6, o Argo CD corrigiu sozinho? Por quê?
4. Onde o Argo CD guarda qual commit está aplicado no cluster?
