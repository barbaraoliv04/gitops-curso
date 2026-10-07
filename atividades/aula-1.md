# Prática 1 — O deploy manual e seus limites

**Aula 1 · 07/10 · 35 minutos**

Objetivo: publicar a aplicação com `kubectl`, provocar uma diferença entre o Git e o cluster
e perceber por que precisamos de GitOps.

## Passo 1 — Seu fork

1. No GitHub, abra o repositório do curso (link enviado pelo instrutor) e clique em **Fork**.
2. Clone o **seu** fork:

```bash
git clone https://github.com/<seu-usuario>/gitops-curso.git
cd gitops-curso
```

## Passo 2 — Ambiente

Confira se o Kubernetes do Docker Desktop está ligado (indicador verde), rode os comandos abaixo e anote o resultado:

```bash
git --version
docker info
kubectl config use-context docker-desktop
kubectl config current-context
kubectl get nodes
git config user.name
git config user.email
```

- O contexto atual deve ser `docker-desktop`.
- O node deve aparecer como `Ready`.
- `git config` deve mostrar o seu nome e e-mail. Se aparecer vazio, configure com
  `git config --global user.name "Seu Nome"` e `git config --global user.email "seu@email"`.

## Passo 3 — Deploy manual

```bash
kubectl create namespace curso
kubectl apply -k apps/web/base -n curso
kubectl get deploy,pods,svc -n curso
```

Abra a aplicação (deixe o comando rodando e use outro terminal para o resto):

```bash
kubectl port-forward svc/web -n curso 8081:80
```

Acesse http://localhost:8081 — deve aparecer **Ambiente: BASE** e **Versao: v1**.

## Passo 4 — Alguém mexeu no cluster

Simule um colega "resolvendo rápido" direto no cluster:

```bash
kubectl scale deploy/web -n curso --replicas=5
kubectl get pods -n curso
```

Agora compare o cluster com o que está no Git:

```bash
kubectl diff -k apps/web/base -n curso
```

## Passo 5 — Mudança só no Git

1. Edite `apps/web/base/index.html` e troque `Versao: v1` por `Versao: v2`.
2. Grave e envie:

```bash
git add apps/web/base/index.html
git commit -m "Página versão v2"
git push
```

3. Recarregue http://localhost:8081. A página mudou?
4. Aplique manualmente e recarregue de novo:

```bash
kubectl apply -k apps/web/base -n curso
```

## Evidência de conclusão

- Página mostrando **Versao: v2**.
- Commit "Página versão v2" visível no seu fork no GitHub.
- Respostas às perguntas abaixo anotadas.

## Perguntas para pensar

1. Depois do passo 4, quantas réplicas o Git dizia que deveriam existir? E quantas existiam?
2. Quem ficou sabendo da mudança de réplicas? Ficou algum registro?
3. No passo 5, por que a página não mudou depois do `git push`?
4. Ao rodar `kubectl apply` no fim, o que aconteceu com as 5 réplicas? Por quê?
5. Se 10 pessoas tivessem acesso ao cluster, como você saberia qual é a versão "certa"?

> Não apague nada: na aula 2 o Argo CD vai assumir estes mesmos recursos.
