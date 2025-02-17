# Instalação do ArgoCD em um Cluster Kubernetes com Raspberry Pi

## Pré-requisitos

- Cluster Kubernetes configurado com pelo menos duas Raspberry Pi
- `kubectl` configurado para se conectar ao cluster

## Passos para Instalação

### 1. Criar o Namespace

Crie o namespace `argocd`:

```sh
kubectl create namespace argocd
```

### 1. Instalar o ArgoCD

Execute o seguinte comando para instalar o ArgoCD no namespace `argocd`:

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Verificar os Pods do ArgoCD

Verifique se todos os pods estão em execução:

```sh
kubectl get pods -n argocd
```

### 3. Expor o Serviço ArgoCD Server

Para acessar a interface web do ArgoCD, exponha o serviço `argocd-server`:

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 4. Obter a Senha Inicial

A senha inicial do usuário `admin` é o nome do pod `argocd-server`. Obtenha a senha com o comando:

```sh
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```

### 5. Acessar a Interface Web

Abra seu navegador e acesse `https://localhost:8080`. Use `admin` como nome de usuário e a senha obtida no passo anterior.

### 6. Alterar a Senha do Admin

Para alterar a senha do usuário `admin`, execute:

```sh
argocd login localhost:8080
argocd account update-password
```

Agora você pode começar a usar o ArgoCD para gerenciar suas aplicações no cluster Kubernetes.
