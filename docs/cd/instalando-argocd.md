# Instalando o ArgoCD no AWS Learner Lab

Neste capítulo, vamos instalar o ArgoCD no cluster k3s que configuramos no AWS Learner Lab.

## O que vamos fazer

- Criar o namespace do ArgoCD
- Instalar os componentes oficiais
- Liberar NetworkPolicy do repo-server
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

## 2. Liberar NetworkPolicy para o repo-server

Em ambientes com políticas de rede mais restritivas (por exemplo, com `default deny`), o `argocd-repo-server` pode precisar de uma regra explícita para receber conexões internas necessárias ao funcionamento do ArgoCD.

Crie o arquivo `argocd-repo-server-network-policy.yaml` com o conteúdo abaixo:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: argocd-repo-server-network-policy
    namespace: argocd
spec:
    podSelector:
        matchLabels:
            app.kubernetes.io/name: argocd-repo-server
    policyTypes:
        - Ingress
    ingress:
        - from:
              - podSelector: {}
          ports:
              - protocol: TCP
                port: 8081
              - protocol: TCP
                port: 8084
```

Aplicar a política:

```bash
kubectl apply -f argocd-repo-server-network-policy.yaml
```

Validar se ela foi criada:

```bash
kubectl get networkpolicy -n argocd
kubectl describe networkpolicy argocd-repo-server-network-policy -n argocd
```

Por que isso é necessário:

- Garante que o `argocd-repo-server` aceite tráfego interno do namespace `argocd` nas portas `8081` e `8084`.
- Evita bloqueios de comunicação entre componentes do ArgoCD em clusters com restrições de rede.

Limite importante desta política:

- Esta regra é de **Ingress** (entrada) e não libera saída para a internet.
- O acesso a repositórios do GitHub depende de conectividade de saída (Egress), DNS e regras de rede do cluster.

!!! note "Pré-requisito"
    A `NetworkPolicy` só terá efeito se o cluster usar um plugin/CNI com suporte a enforcement de políticas (como Calico, Cilium ou kube-router).

## 3. Acessar o ArgoCD via port-forward

Como estamos em ambiente de laboratório, a forma mais simples é encaminhar porta local:

```bash
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8080:443
```

Com isso, acesse no navegador:

- <https://SEU-IP-EC2:8080>

Se você estiver com túnel SSH local, também pode usar:

- <https://localhost:8080>

## 4. Obter usuário e senha iniciais

O usuário padrão é `admin`.

A senha inicial pode ser lida do secret:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d; echo
```

Após o primeiro login, altere a senha na interface.

!!! warning "Ambiente de aula não é produção"
    Em produção, você deve publicar o ArgoCD com Ingress, TLS válido e controle de acesso adequado.

## 5. Verificação rápida

Confira se o servidor está pronto:

```bash
kubectl get svc -n argocd
kubectl get deployments -n argocd
```

Se o serviço `argocd-server` existir e os deployments estiverem `AVAILABLE`, podemos avançar para o repositório GitOps.

## 6. (Opcional) CLI do ArgoCD

Você pode usar também a CLI:

```bash
argocd login localhost:8080 --username admin --password SUA_SENHA --insecure
argocd app list
```

A CLI facilita automações e troubleshooting, mas nesta aula focaremos primeiro na interface web para visualização didática do fluxo.
