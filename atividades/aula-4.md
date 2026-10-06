# Prática 4 — App of Apps

**Aula 4 · 15/10 · 10 minutos (antes do desafio final)**

Objetivo: deixar uma Application "raiz" gerenciando as Applications dos ambientes,
para que até a criação de um ambiente novo aconteça por commit.

## Passo 1 — Criar a raiz

```bash
cat argocd/root.yaml
kubectl apply -f argocd/root.yaml
```

Na interface, abra **root**. A árvore deve mostrar as Applications **web-dev** e **web-prod**
como recursos filhos.

## Passo 2 — Testar a regra

Apague uma Application "na mão" e veja o que acontece:

```bash
kubectl delete application web-dev -n argocd --wait=false
kubectl get applications -n argocd -w     # Ctrl+C para sair
```

## Evidência de conclusão

- **root**, **web-dev** e **web-prod** em **Synced** e **Healthy**.
- `web-dev` recriada automaticamente após ser apagada.

## Perguntas de checagem

1. Depois deste passo, qual é o jeito certo de criar um ambiente novo?
2. O que aconteceria se você apagasse `argocd/apps/web-prod.yaml` do Git e fizesse push? Por quê?

Agora siga para o [desafio final](desafio-final.md).
