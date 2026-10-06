# Curso GitOps com Argo CD — Laboratório

Repositório das práticas do curso **GitOps — Fundamentos e prática** (Minsait, outubro de 2026).

Cada aluno trabalha no **próprio fork** deste repositório. O Argo CD do seu cluster
local (Kubernetes do Docker Desktop) vai ler o seu fork e manter o cluster igual ao que está no Git.

## Encontros

| Aula | Data | Tema | Atividade |
|------|------|------|-----------|
| 1 | 07/10 (qua), 17h–19h | Fundamentos: Git, Kubernetes e o deploy manual | [atividades/aula-1.md](atividades/aula-1.md) |
| 2 | 08/10 (qui), 17h–19h | Argo CD e a primeira Application | [atividades/aula-2.md](atividades/aula-2.md) |
| 3 | 14/10 (qua), 17h–19h | Sync automático, drift e ambientes com Kustomize | [atividades/aula-3.md](atividades/aula-3.md) |
| 4 | 15/10 (qui), 17h–19h | App of Apps, boas práticas e desafio final | [atividades/aula-4.md](atividades/aula-4.md) e [atividades/desafio-final.md](atividades/desafio-final.md) |

Antes da primeira aula, siga o [PRE-REQUISITOS.md](PRE-REQUISITOS.md).

## Estrutura

```
apps/web/
  base/                 Deployment, Service e página (index.html) da aplicação
  overlays/dev/         1 réplica, namespace web-dev, página DEV
  overlays/prod/        3 réplicas, namespace web-prod, página PROD
argocd/
  aula2/web.yaml        primeira Application, com sync manual
  apps/                 Applications de cada ambiente (sync automático)
  root.yaml             App of Apps: gerencia tudo que está em argocd/apps
atividades/             roteiros das práticas
```

## Portas usadas no curso

| Porta local | O que abre | Comando |
|-------------|-----------|---------|
| 8080 | Interface do Argo CD | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| 8081 | Aplicação (aula 1 e 2, namespace curso) | `kubectl port-forward svc/web -n curso 8081:80` |
| 8082 | Aplicação em dev | `kubectl port-forward svc/web -n web-dev 8082:80` |
| 8083 | Aplicação em prod | `kubectl port-forward svc/web -n web-prod 8083:80` |

## A regra de ouro

> A partir da aula 2, **toda mudança na aplicação passa pelo Git**: editar, `git commit`, `git push`.
> O `kubectl` serve para observar, não para alterar.
