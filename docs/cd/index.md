# Entrega Contínua

Já estudamos um pouco sobre a integração contínua, onde o código é integrado e testado automaticamente. Agora, vamos falar sobre a entrega contínua.

A entrega contínua é uma prática de desenvolvimento de software onde as alterações no código são automaticamente preparadas para um lançamento em produção. Isso significa que, após cada alteração no código, o software é automaticamente testado e preparado para ser lançado, permitindo que novas funcionalidades sejam disponibilizadas rapidamente e com menos riscos.

Uma técnica muito comum na aplicação da entrega contínua é o GitOps, que é uma combinação de práticas de desenvolvimento ágil e operações de TI. O GitOps utiliza o Git como a única fonte de verdade para a infraestrutura e as aplicações, permitindo que as equipes de desenvolvimento e operações trabalhem juntas de forma mais eficiente.

Ainda, podemos utilizar o GitOps para automatizar o processo de entrega contínua, permitindo que as alterações no código sejam automaticamente implantadas em ambientes de produção. Isso reduz o tempo e o esforço necessários para implantar novas funcionalidades e correções de bugs, além de aumentar a confiabilidade e a segurança do processo de entrega.

Integrado ao ambiente de k8s que estamos utilizando, o GitOps pode ser utilizado para gerenciar a infraestrutura e as aplicações em contêineres. Isso permite que as equipes de desenvolvimento e operações trabalhem juntas de forma mais eficiente, utilizando práticas ágeis e automação para implantar novas funcionalidades e correções de bugs rapidamente.

Várias ferramentas podem ser utilizadas para implementar o GitOps, como o ArgoCD e o Flux. Essas ferramentas permitem que as equipes de desenvolvimento e operações trabalhem juntas de forma mais eficiente, utilizando práticas ágeis e automação para implantar novas funcionalidades e correções de bugs rapidamente. Nas nossas aulas faremos uso do ArgoCD, que é uma ferramenta de entrega contínua para Kubernetes que permite implantar aplicações de forma automatizada e segura, e utiliza o Git como a única fonte de verdade para a infraestrutura e as aplicações.

## Instalação do ArgoCD

Para instalar o ArgoCD, você pode usar o seguinte comando:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Isso criará um namespace chamado `argocd` e instalará o ArgoCD nesse namespace.
Após a instalação, você pode acessar o ArgoCD através do seguinte comando:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Isso fará o encaminhamento da porta 8080 do seu computador local para a porta 443 do serviço `argocd-server` no namespace `argocd`. Você pode acessar o ArgoCD em `https://localhost:8080`.
O nome de usuário padrão é `admin` e a senha pode ser obtida com o seguinte comando:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d; echo
```

Isso exibirá a senha inicial do usuário `admin`.

Agora já estamos preparados para utilizar o ArgoCD e implementar a entrega contínua em nossos projetos. Vamos ver como fazer isso na prática.

Mas, antes, vamos preparar o repositório do nosso projeto para que possamos utilizar o ArgoCD. Para isso, vamos criar um repositório no GitHub e adicionar os arquivos de configuração do ArgoCD.

## Preparação do repositório

Vamos continuar usando o projeto que criamos na aula anterior (orquestração de contêineres). Para isso, vamos criar um repositório no GitHub e adicionar os arquivos de configuração do ArgoCD.

Para tal suba o seu projeto para o GitHub, e crie um repositório, por exemplo chamado `hello-fastapi-k8s` (esta é apenas uma sugestão). Após subir a primeira versão para o repostório do GitHub, já sugiro que você crie os dois segredos no GitHub para fazer o upload da imagem do Docker para o Docker Hub e do arquivo de configuração do ArgoCD para o repositório do GitHub.

Para compatibilidade com o que mostraremos nos arquivos de configuração do _workflow_ do GitHub, sugiro que você crie esses dois segredos com os seguintes nomes:

- `DOCKERHUB_USERNAME`: seu nome de usuário do Docker Hub
- `DOCKERHUB_TOKEN`: seu token de acesso do Docker Hub

O `DOCKERHUB_TOKEN` pode ser gerado na sua conta do Docker Hub, na seção de configurações de segurança.

Também é necessário permitir que o GitHub Actions faça push no repositório. Para isso, você deve acessar as configurações do seu repositório no GitHub, clicar em `Actions` e depois em `General`. Na seção `Workflow permissions`, selecione a opção `Read and write permissions`. Isso permitirá que o GitHub Actions faça push no repositório.

Agora, vamos criar o arquivo de configuração do fluxo de CI/CD do GitHub. Para isso, crie uma pasta chamada `.github` na raiz do seu repositório e dentro dela crie outra pasta chamada `workflows`. Dentro da pasta `workflows`, crie um arquivo chamado `ci-cd.yml`. O caminho completo do arquivo deve ser `.github/workflows/ci-cd.yml`. Esse arquivo será responsável por configurar o fluxo de CI/CD do GitHub. O conteúdo do arquivo deve ser o seguinte:

```yaml title="./.github/workflows/ci-cd.yml" linenums="1"
name: CICD

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            eduardosilvasc/hello-fastapi-k8s:latest
            eduardosilvasc/hello-fastapi-k8s:${{ github.sha }}

      - name: Setup Kustomize
        uses: imranismail/setup-kustomize@v2

      - name: Update kustomization.yaml
        run: |
          cd k8s
          kustomize edit set image hello-fastapi=eduardosilvasc/hello-fastapi-k8s:$GITHUB_SHA

      - name: Commit changes
        run: |
          git config --local user.name "GitHub Actions"
          git config --local user.email "actions@github.com"
          git commit -am "Update kustomization.yaml with new image"

      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.ref }}
```

Note que eu fixei em alguns locais o nome da imagem como `eduardosilvasc/hello-fastapi-k8s`, que é o nome do repositório que eu criei no Docker Hub. Você deve alterar esse nome para o nome do repositório que você criou no Docker Hub, e também o nome da imagem que você utilizará. Esses nomes devem ser os mesmos que você utilizará no arquivo de configuração do ArgoCD, que veremos a seguir.

## Configuração do Kustomize

Agora, vamos criar o arquivo de configuração do Kustomize. Para isso, crie uma pasta chamada `k8s` na raiz do seu repositório e dentro dela crie um arquivo chamado `kustomization.yaml`. O caminho completo do arquivo deve ser `k8s/kustomization.yaml`. Esse arquivo será responsável por configurar o Kustomize. O conteúdo do arquivo deve ser o seguinte:

```yaml title="./k8s/kustomization.yaml" linenums="1"
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
images:
  - name: hello-fastapi
    newName: eduardosilvasc/hello-fastapi-k8s
    newTag: c58b625834eb8b45c0d3d3b45ba4cbc288fc5146
```

No que, embora na pasta também tenhamos o arquivo `pod.yaml`, não o colocamos no `kustomization.yaml` porque ele não é necessário para o ArgoCD. O ArgoCD irá criar o pod automaticamente a partir do deployment.

Além disso, perceba que na declaração de `images` temos o parâmetro `name` com nome que está no nome do template do `deployment.yaml`, e o parâmetro `newName` com o nome da imagem que você subiu para o Docker Hub. O parâmetro `newTag` deve ser o mesmo que você utilizará no arquivo de configuração do ArgoCD, que veremos a seguir.

Com essa configuração feita, você já pode subir a sua atualização para o repositório do GitHub. O fluxo de CI/CD do GitHub irá executar automaticamente o processo de build e push da imagem para o Docker Hub, e atualizar o arquivo `kustomization.yaml` com a nova tag da imagem. Isso significa que, sempre que você fizer uma alteração no código e subir para o repositório do GitHub, a imagem será atualizada automaticamente no Docker Hub.

## Configuração do ArgoCD

Agora, vamos acessar o ArgoCD e criar um novo aplicativo. Para isso, acesse o ArgoCD em `https://localhost:8080` e faça login com o usuário `admin` e a senha que você obteve anteriormente.

Após fazer login, você verá a tela inicial do ArgoCD. Clique no botão `New App` para criar um novo aplicativo.

Na tela de criação do aplicativo, preencha os campos da seguinte forma:

- **Application Name**: hello-fastapi-k8s
- **Project**: default
- **Sync Policy**: Manual
- **Repository URL**: https://github.com/eduardo-da-silva/hello-fastapi-k8s.git
- **Revision**: HEAD
- **Path**: k8s
- **Cluster**: https://kubernetes.default.svc
- **Namespace**: default

Com esses dados preenchidos, clique no botão `Create` para criar o aplicativo.

Agora, você verá o aplicativo criado na tela inicial do ArgoCD. Clique no nome do aplicativo para acessar a tela de detalhes do aplicativo. Na tela de detalhes do aplicativo, você verá a árvore de recursos do aplicativo. Clique no botão `Sync` para sincronizar o aplicativo com o repositório do GitHub. Isso irá criar os recursos no Kubernetes a partir do arquivo `kustomization.yaml`.

Após a sincronização, você verá os recursos criados na árvore de recursos do aplicativo. Você pode clicar em cada recurso para ver os detalhes do recurso.

Como estamos executando o ArgoCD dentro de um cluster com o kind, não temos acesso ao serviço que foi implantado com o Kustomize e o ArgoCD. Para acessar o serviço, você pode usar o seguinte comando:

```bash
kubectl port-forward svc/hello-fastapi-service 8000:8000
```

Note que o nome do serviço pode variar de acordo com o que você definiu no arquivo `service.yaml`. O comando acima fará o encaminhamento da porta 8000 do seu computador local para a porta 8000 do serviço. Esse número de porta também pode variar de acordo com o que você definiu no arquivo `service.yaml`, `deployment.yaml` e na própria aplicação FastAPI. Você pode acessar o serviço em `http://localhost:8000`. Isso fará com que você consiga acessar a aplicação FastAPI que foi implantada no Kubernetes.
