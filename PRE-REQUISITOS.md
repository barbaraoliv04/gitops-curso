# Pré-requisitos — prepare antes de 07/10

Reserve cerca de 30 minutos. Se travar em algum passo, envie a mensagem de erro ao instrutor antes da aula.

## 1. Máquina

- 8 GB de RAM no total (o Docker Desktop precisa de pelo menos 4 GB) e 2 CPUs
- 15 GB de disco livre
- Acesso à internet para GitHub, `quay.io`, `ghcr.io`, `public.ecr.aws` e Docker Hub

## 2. Ferramentas

| Ferramenta | Para quê | Como conferir |
|-----------|---------|---------------|
| Git | versionar as mudanças | `git --version` |
| Docker Desktop | executar o cluster Kubernetes local | `docker info` |
| kubectl | conversar com o cluster (já vem com o Docker Desktop) | `kubectl version --client` |
| Editor (VS Code recomendado) | editar YAML | — |
| argocd CLI (opcional) | usar o Argo CD pelo terminal | `argocd version --client` |

Instalação:

- Docker Desktop: https://docs.docker.com/desktop/
- argocd CLI (opcional): https://argo-cd.readthedocs.io/en/stable/cli_installation/

No Windows, use o **Git Bash** ou o **WSL** para os comandos do curso (eles usam sintaxe de terminal Linux).

## 3. Ative o Kubernetes no Docker Desktop

1. Abra o Docker Desktop.
2. Em **Settings → Resources**, deixe pelo menos **4 GB de memória** e **2 CPUs**
   (no Windows com WSL 2, a memória é controlada pelo arquivo `.wslconfig`).
3. Abra a área **Kubernetes** e clique em **Create cluster** (em versões mais antigas:
   **Settings → Kubernetes → Enable Kubernetes**).
4. Escolha o modo **Kubeadm** (um node só, o mais simples) e confirme.
5. Aguarde o indicador do Kubernetes ficar verde no rodapé do Docker Desktop.

No terminal:

```bash
kubectl config use-context docker-desktop
kubectl get nodes
```

O node `docker-desktop` deve aparecer como `Ready`.

> Já tem outro cluster no seu kubectl (do trabalho, por exemplo)? Confira sempre o contexto com
> `kubectl config current-context` antes dos exercícios. O curso deve rodar só em `docker-desktop`.

## 4. Conta no GitHub

1. Crie uma conta em https://github.com (se ainda não tiver).
2. Configure o Git com seu nome e e-mail:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu-email@exemplo.com"
```

3. Para fazer `git push` por HTTPS, gere um **Personal Access Token** (Settings → Developer settings →
   Personal access tokens → Fine-grained tokens) com permissão *Contents: Read and write* no seu fork.
   Use o token como senha quando o Git pedir.

## 5. Pré-baixe as imagens (economiza tempo na aula)

No modo Kubeadm, o cluster usa as mesmas imagens do Docker:

```bash
docker pull nginx:1.27-alpine
docker pull quay.io/argoproj/argocd:v3.5.3
docker pull ghcr.io/dexidp/dex:v2.45.1
docker pull public.ecr.aws/docker/library/redis:8.2.3-alpine
```

## Alternativa: Minikube

Se não puder usar o Kubernetes do Docker Desktop, instale o Minikube
(https://minikube.sigs.k8s.io/docs/start/) e use `minikube start --driver=docker --cpus=2 --memory=4096`.
Todos os exercícios funcionam igual.
