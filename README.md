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


# 4. Dev/Ops AI
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
/skills

grill-me @deployment.yaml
```

## 5.3. Caveman
Instala y usa el skill.
```
Resumen de la implementacion del deployment y estado de los pods
```

## 5.4. Kube Skill
Crea un skill propio para controlar el cluster y sus recursos
- Nuevo folder `.agents`/skills/kube
- Nuevo archivo: `SKILL.md`
```yaml
---
name: kube-powers
description: Herramienta para la administracion de kubernetes
allowed-tools: Read, Grep, Glob, Write, Edit, Bash(kubectl:*), Bash(helm:*), Bash(curl:*), Bash(cat:*), Bash(ls:*), Bash(mkdir:*), Bash(kustomize:*)
---

# Rol y Comportamiento
Eres un asistente experto en Kubernetes y SRE. Tu objetivo es ayudar a diagnosticar y solucionar problemas en el cluster.
## Reglas de operación:
1. **Inspección primero:** Antes de proponer cambios, consulta el estado actual con `kubectl get pods`, `kubectl describe`, o `kubectl logs`. También analiza si existe un `limitRange` o `ResourceQuota` que esté impidiendo la creación de los recursos.
2. **Explica la causa raíz:** Explica de forma clara y sencilla qué error ocurrió (ej. falta de recursos, CrashLoopBackOff, ImagePullBackOff).
3. **Seguridad:** NUNCA ejecutes `kubectl delete` sin pedir confirmación explícita al usuario.
4. **Propón soluciones:** Muestra el manifiesto corregido antes de aplicarlo.
```
Alternativa mas segura para `allowed-tools`: sustituye `*` por comandos especificos.
```yaml
allowed-tools: Read, Grep, Glob, Write, Edit, Bash(kubectl get:*), Bash(kubectl describe:*), Bash(kubectl logs:*), Bash(kubectl apply:*), Bash(kubectl diff:*), Bash(kubectl explain:*), Bash(helm list:*), Bash(helm status:*), Bash(curl:*), Bash(cat:*), Bash(ls:*)
```
Reinicia Opencode y prueba el skill con el siguiente prompt:
```
/skill
kube-powers Resuelve el error de implementacion del deployment
```
### 5.4.1. Elimina y prueba
Elimina el `deployment`
```sh
kubectl delete -f deployment.yaml
```
Y prueba de nuevo la creación del recurso ahora usando el `skill` tomando el requerimiento del punto 3.
Conserva los errores de los recursos.
```
/skill
kube-powers @deployment.yaml implementa
```

# 6. Agentes
## 6.1. Agents.md
Instrucciones y directivas globales que todo agente de IA debe seguir dentro de este repositorio sin necesidad de recordárselo en cada prompt.

### 6.1.1. Ejemplo: `AGENTS.md` en la raíz del proyecto
```markdown
# AI Directives for aiOps Project
## Convenciones de Kubernetes
- Todos los recursos de laboratorio deben crearse dentro del namespace `dev`.
- Nunca uses el namespace `default` ni `kube-system`.
- Todo contenedor debe tener declarados `requests` y `limits` de CPU y memoria.
## Comandos habituales
- Validar estado general: `kubectl get pods -n dev`
- Ver cuotas del namespace: `kubectl get quota,limitrange -n dev`
## Reglas de Seguridad
- Prohibido ejecutar `kubectl delete ns` o eliminar CRDs compartidos.
```

## 6.2. Built-in
- `Build`
- `Plan`
- `@` para sub-agentes: `Explore`

## 6.3. Contruye un agente
- Nuevo folder `.opencode`/agents
- archivo: `kube.md`

```yaml
---
name: kube
description: Agente SRE autónomo para despliegue y diagnóstico en Kubernetes
mode: primary
tools:
  read: true
  grep: true
  glob: true
  task: true
  bash: true
  write: true
  edit: true
permission:
  edit: allow
  bash:
    "*": allow
    "kubectl delete*": ask
    "helm uninstall*": ask
    "rm *": ask
---

# Rol: Kubernetes SRE Specialist

Eres un Ingeniero DevOps - SRE de Storylabs.

## Responsabilidad:
- Si el usuario te pide diseñar o desplegar una nueva arquitectura/recurso ambiguo, usa `grill-me` para clarificar requerimientos (puertos, dominios, réplicas) antes de aplicar.
- Utiliza `kube-powers` para la inspección, validación contra cuotas y aplicación en el clúster.
- Mantén la comunicación clara, técnica y concisa (`caveman`).

```
## 6.4. Probar el agente
Reinicia Opencode y busca el agente `TAB`

### 6.4.1. Prueba de secuencia
Pregunta:
```
Si le doy una instruccion cuales son sus pasos a seguir?
```
### 6.4.2. Prueba de implementacion
Repetir el caso 3.
```sh
@deploymtment.yaml implementa
```

## 6.5. Analiza: Que es un skill y agente??

# 7. Otros skills
- [Codebase Memory](https://github.com/DeusData/codebase-memory-mcp)
- [Archify](https://github.com/tt-a1i/archify)

## 7.1. Probar Codebase Memory
```
/skill
codebase memory mcp indexa
cuales yaml quedaron indexados?
cuales beneficios obtuvimos al tener yaml de kuberentes indexados?
tenemos alguna ganancia en cuanto a uso de tokens?
```
## 7.2. Probar archify
```sh
/skill
archify de los recursos de kubernetes instalados en el cluster
```
### Para ver el diagrama
```sh
python3 -m http.server 8080 -b 0.0.0.0 -d ~/ai-ops
```
# 8. MCP

## 8.1. Configuración inicial
```sh
# Comprobar la ruta del kubeconfig
cat /home/azureuser/.kube/config
# editar el archivo de config de Opencode
vim ~/.config/opencode/opencode.json
```
```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "kubernetes": {
      "type": "local",
      "command": ["npx", "-y", "mcp-server-kubernetes"],
      "environment": {
        "KUBECONFIG": "/home/azureuser/.kube/config"
      },
      "enabled": true
    }
  }
}
```

### 8.1.1. Validar servidor MCP
```sh
opencode mcp list
opencode debug config
```

### 8.1.2. Preguntas de exploración en OpenCode
```
¿Cuáles herramientas de Kubernetes tienes disponibles a través de MCP?
```

## 8.2. Prueba de diagnóstico de incidentes
Aplica un pod intencionalmente dañado:
```sh
kubectl apply -f assets/broken-app.yaml
```
Prompt para el agente:
```
Inspecciona los pods del namespace dev mediante tus herramientas de Kubernetes.

Si encuentras algún pod fallando, analiza sus eventos/logs y explícame la causa raíz.
```

## 8.3. MCP Seguro (Enterprise Hardening)

```
¿Este MCP es seguro para ejecutarlo en un entorno empresarial de producción?
```

### 8.3.1. ¿Por qué la configuración anterior no es segura?
- **Namespace no restringido:** El servidor por defecto puede exponer `k8s://default/*`, violando las directivas de `AGENTS.md` (donde solo se permite `dev`).
- **Herramientas destructivas expuestas:** Habilita sin control `kubectl_generic`, `exec`, `delete`, `patch`, y `helm install`.
- **Exposición de secretos:** Puede exponer valores en texto plano de Secrets y ConfigMaps.

### 8.3.2. Configuración Segura para OpenCode
Cambia a `kubernetes-mcp-server` (nativo API, con *secret masking* + *non-destructive* + RBAC) y limita los permisos en OpenCode:

`~/.config/opencode/opencode.json`:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "k8s-safe": {
      "type": "local",
      "command": ["npx", "-y", "kubernetes-mcp-server", "--toolsets=config,core", "--non-destructive"],
      "environment": {
        "KUBECONFIG": "/home/azureuser/.kube/config"
      },
      "enabled": true
    }
  },
  "permission": {
    "*": "ask",
    "k8s-safe_*": "ask",
    "k8s-safe_kubectl_delete*": "deny",
    "k8s-safe_kubectl_apply*": "deny",
    "k8s-safe_kubectl_exec*": "deny",
    "k8s-safe_helm_*": "deny"
  }
}
```

### 8.3.3. Probar la configuración segura
Reinicia OpenCode y valida que las operaciones destructivas o de mutación estén bloqueadas (`deny`), permitiendo solo lectura diagnóstica (`core` + `config` no destructivo).

## 8.4. Avanzado con Kubernetes guards:
- Kyverno / OPA Gatekeeper
- polaris
- kubeArmor
- Falco

buscar skills para kube y tf
