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

## 2. Liberar a comunicação no Security Group

Como existem vários nós (pelo menos um 'master' e um 'worker') é necessário liberar a comunicação entre eles para o ArgoCD funcionar corretamente. Para isso, siga os passos:

1. Acesse o console da AWS e vá para a seção de EC2.
2. Clique em "Security Groups" no menu lateral.
3. Encontre o Security Group associado às suas instâncias do laboratório (geralmente tem
    um nome relacionado ao laboratório ou ao cluster).
4. Clique no Security Group para abrir os detalhes.
5. Vá para a aba "Inbound rules" e clique em "Edit inbound rules".
6. Adicione uma nova regra com as seguintes configurações:
    - Type: All traffic
    - Protocol: All
    - Port Range: All
    - Source: Custom (selecione o mesmo Security Group para permitir a comunicação entre os hosts)
7. Salve as regras.

Essa configuração é necessária para que os componentes do ArgoCD possam se comunicar entre si, especialmente o repo-server, que precisa acessar o repositório Git e outros serviços. Outros componentes, como o server e o application-controller, também dependem dessa comunicação para funcionar corretamente. Detectar problemas de comunicação entre os nós pode ser difícil, então liberar toda a comunicação facilita o processo de instalação e configuração do ArgoCD no ambiente de laboratório.

!!!warning "Atenção"

    Como essa é uma configuração de laboratório, estamos liberando toda a comunicação entre os hosts para facilitar o processo. Em um ambiente de produção, você deve restringir as regras de acordo com as necessidades específicas do ArgoCD e do seu cluster.

Nos passos seguintes, também vamos disponibilizar uma aplicação FastAPI, via port-forward, usando a porta TCP/8001, então certifique-se de liberar essa porta também, se necessário. Para isso, adicione outra regra de entrada (como feito no exemplo anterior) com as seguintes configurações:

- Type: Custom TCP
- Protocol: TCP
- Port Range: 8001
- Source: Anywhere (0.0.0.0/0)

Com essa regra, você poderá acessar a aplicação FastAPI que será implantada posteriormente, mesmo que esteja usando um túnel SSH local para acessar o cluster. Lembre-se de que, em um ambiente de produção, é importante restringir o acesso a portas específicas apenas para os endereços IP ou redes necessárias, em vez de permitir acesso irrestrito.

!!!warning "Para testes e debug"
    
    Se você estiver enfrentando problemas de comunicação entre os componentes do ArgoCD, pode ser útil liberar temporariamente toda a comunicação entre os hosts para facilitar o processo de diagnóstico. No entanto, lembre-se de que essa configuração é apenas para fins de teste e não deve ser usada em ambientes de produção. Após resolver os problemas, certifique-se de restringir as regras de segurança adequadamente para proteger seu ambiente.

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
