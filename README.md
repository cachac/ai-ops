# ai-ops <!-- omit in toc -->

# 1. Intro
1 - Qué tan dicil perciben de implementar AI?
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
```sh
cd ~
git clone https://github.com/cachac/ai-ops.git
cd ai-ops

kubectl create ns dev
kubectl apply -f assets/quota.yaml
```

# 3. Vibe Coding "Chat prompt"
Prompt generico en cualquier chat
```
crea un deployment con los siguientes requerimientos:
nombre del deployment kubelabs
namespace dev
replicas 2
imagen: cachac/kubelabs:3.0
recursos:
- request:
	- 200m CPU
	- 512Mi Memoria
Puerto: 8080
```
## 3.1. Ejecutar el deployment
Crea el archivo `deployment.yaml` con el contenido generado.
```sh
kubectl apply -f deployment.yaml
kubectl get pods -n dev
kubectl get rs -n dev
kubectl describe rs -n dev
```

- El `ReplicaSet` falla al crear los `pods` (`FailedCreate`) porque los recursos solicitados (`requests: 200m CPU / 512Mi`) exceden el límite máximo permitido por el `LimitRange` (`max: 100m CPU / 256Mi`) en el namespace `dev`.

## 3.2. Pedir al chat que corrija el problema.
El chat falló al inicio porque no conoce la infraestructura y configuracion.


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
Permisos*
```

# 5. Skills básicos
- [skills.sh](https://www.skills.sh/)
## 5.1. Grill Me
```sh
sudo apt install npm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.7/install.sh | bash
nvm list
# debe tener instlado al menos la versión 22

# en caso de tener una versión anterior, instalar y usar 22
nvm install 22

npx skills add https://github.com/mattpocock/skills --skill grill-me
cat  ~/.agents/skills/grill-me/SKILL.md
```
Reiniciar la TUI para tomar el skill.

## 5.2. Uso del skill
Primero ubicar la carpeta
```sh
!pwd
```
Usa el skill para implementar el `deployment.yaml` del paso 3.
```sh
/skills - grill me
```

## 5.3. Caveman
Instala y usa el skill.
```
Resumen de la implementacion del deployment y estado de los pods
```





-- ideas a desarrollar
codebase memory mcp
archify
agente para kubernetes
agente para terraform
mcp

## 5.4. avanzado con kubernetes guards:
- Kyverno / OPA Gatekeeper
- polaris
- kubeArmor
- Falco

buscar skills para kube y tf
