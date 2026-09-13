# ai-ops <!-- omit in toc -->

# 1. Intro
1 - Qué tan dicil persiben de implementar AI?
conceptos:
- LLM: Fabel, Astra, Grok, Qwen, Minimax, Gemini, K3, GLM, etc
- prompts
- Agentes
- RAG
- skills
- MCP
- Loops
- Hooks
- TUI
- Harness

# 2. Instalacion
## 2.1. Instalación del cluster K3s
```sh
sudo apt update
sudo apt upgrade -y
ufw disable

curl -sfL https://get.k3s.io | sh -

sudo systemctl status k3s

sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

kubectl get nodes
```
Para remover el cluster: sudo /usr/local/bin/k3s-uninstall.sh

## 2.2. Instalar Helm
```sh
sudo snap install helm --classic
```

## 2.3. Instalar Ingress Controller
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

kubectl create ns ingress-nginx

helm upgrade -i ingress-nginx ingress-nginx/ingress-nginx \
--namespace ingress-nginx \
--set controller.metrics.enabled=true \
--set controller.podAnnotations."prometheus\.io/scrape"=true \
--set controller.podAnnotations."prometheus\.io/port"=10254
```


## 2.4. Validar la instalación
```sh
helm list -A
```
Resultado:
```
ingress-nginx   ingress-nginx   1 deployed
```
### 2.4.1. En caso de existir otras instalaciones como Traefik se deben eliminar
```sh
helm uninstall traefik -n kube-system
helm uninstall traefik-crd -n kube-system
```

## 2.5. Quotas + Limitranges

# 3. Vibe Coding "Chat prompt"
Prompt generico en cualquier chat
```
crea un deployment con los siguientes requerimientos:
nombre del deployment kubelabs
namespace aiops
replicas 2
imagen: cachac/kubelabs:3.0
recursos:
- 20m CPU
- 128Mi Memoria
Puerto: 8080
```
## 3.1. Ejecutar el deployment
Crea el archivo `deployment.yaml` con el contenido generado.
```sh
kubectl apply -f deployment.yaml
kubectl get pods -n aiops
```

# 4. Agentes
## 4.1. OpenCode
- [Instalar](https://opencode.ai/download)
```sh
curl -fsSL https://opencode.ai/install | bash
```
### 4.1.1. Reiniciar la terminal y abrir
```sh
opencode
```
#### 4.1.1.1. comandos
```sh
/models
/variants
/connect
```

# 5. Skills básicos



-- ideas a desarrollar
grill-me
caveman
codebase memory mcp
archify
agente para kubernetes
agente para terraform

## 5.1. avanzado con kubernetes guards:
- Kyverno / OPA Gatekeeper
- polaris
- kubeArmor
- Falco

buscar skills para kube y tf
