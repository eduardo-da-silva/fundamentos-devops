# Instalando o ArgoCD no AWS Learner Lab

Neste capítulo, vamos instalar o ArgoCD no cluster k3s que configuramos no AWS Learner Lab.

## O que vamos fazer

- Criar o namespace do ArgoCD
- Instalar os componentes oficiais
- Acessar a interface web
- Obter credenciais iniciais

## 1. Instalar o ArgoCD no cluster

Execute os comandos abaixo na máquina onde seu `kubectl` está configurado para o cluster do laboratório:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Acompanhe os pods até todos ficarem em `Running`:

```bash
kubectl get pods -n argocd -w
```

## 2. Acessar o ArgoCD via port-forward

Como estamos em ambiente de laboratório, a forma mais simples é encaminhar porta local:

```bash
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8080:443
```

Com isso, acesse no navegador:

- <https://SEU-IP-EC2:8080>

Se você estiver com túnel SSH local, também pode usar:

- <https://localhost:8080>

## 3. Obter usuário e senha iniciais

O usuário padrão é `admin`.

A senha inicial pode ser lida do secret:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d; echo
```

Após o primeiro login, altere a senha na interface.

!!! warning "Ambiente de aula não é produção"
    Em produção, você deve publicar o ArgoCD com Ingress, TLS válido e controle de acesso adequado.

## 4. Verificação rápida

Confira se o servidor está pronto:

```bash
kubectl get svc -n argocd
kubectl get deployments -n argocd
```

Se o serviço `argocd-server` existir e os deployments estiverem `AVAILABLE`, podemos avançar para o repositório GitOps.

## 5. (Opcional) CLI do ArgoCD

Você pode usar também a CLI:

```bash
argocd login localhost:8080 --username admin --password SUA_SENHA --insecure
argocd app list
```

A CLI facilita automações e troubleshooting, mas nesta aula focaremos primeiro na interface web para visualização didática do fluxo.
