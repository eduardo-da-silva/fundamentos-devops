## Automatizando o upload da imagem Docker para o DockerHub

Nesta etapa, vamos automatizar o upload da imagem Docker para o DockerHub sempre que houver um push no repositório. Isso é útil para garantir que a imagem mais recente esteja sempre disponível em um registro de contêiner, facilitando a implantação em diferentes ambientes.

Para isso, vamos usar o GitHub Actions para criar um pipeline de CI/CD que constrói a imagem Docker e faz o upload para o DockerHub. O pipeline será acionado sempre que houver um push no repositório, garantindo que a imagem mais recente esteja sempre disponível.

### Configurando o DockerHub

O DockerHub é um serviço de registro de contêineres que permite armazenar e compartilhar imagens Docker. Depois que temos um container gerado, no processo de DevOps, é comum fazer o upload da imagem para um registro de contêineres, como o DockerHub, para facilitar a distribuição e implantação da aplicação em diferentes ambientes.

Para isso, é necessário que tenhamos uma conta no DockerHub. Caso não tenha, você pode criar uma conta gratuita em [DockerHub](https://hub.docker.com/). Como a conta criada, você poderia fazer o upload da imagem usando o comando `docker push`, mas como o objetivo é fazer isso de forma automatizada, vamos configurar o GitHub Actions para fazer o upload da imagem Docker para o DockerHub sempre que houver um push no repositório.

Então, precisamos gerar um token de acesso no DockerHub para permitir que o GitHub Actions faça o upload da imagem. Para isso, siga os passos abaixo:

1. Acesse sua conta no DockerHub.
2. Clique na sua foto de perfil no canto superior direito e selecione "Account Settings".
3. Na seção "Personal Access Tokens", clique em "Generate New Token".
4. Dê um nome para o token, defina um prazo de expiração e as permissões de acesso (recomendado "Read, Write, and Delete").
5. Clique em "Generate" e copie o token gerado. Guarde-o em um local seguro, pois você não poderá vê-lo novamente.

Nas próximas etapas, vamos configurar o GitHub Actions para fazer o upload da imagem Docker para o DockerHub usando esse token de acesso.

### Guardando o token no GitHub

Agora que temos o token de acesso do DockerHub, precisamos armazená-lo como um segredo no repositório do GitHub. Isso permitirá que o GitHub Actions acesse o token de forma segura durante a execução do pipeline.

Para armazenar o token como um segredo no GitHub, siga os passos abaixo:

1. Acesse o repositório do GitHub onde você configurou o GitHub Actions.
2. Clique na aba "Settings" (Configurações) do repositório.
3. No menu lateral, clique em "Secrets and variables" e depois em "Actions".
4. Clique no botão "New repository secret" (Novo segredo do repositório).
5. Dê um nome para o segredo, como `DOCKERHUB_TOKEN`, e cole o token de acesso que você gerou no DockerHub.
6. Clique em "Add secret" (Adicionar segredo) para salvar o token.
7. Agora, o token estará disponível como um segredo no GitHub Actions e poderá ser usado no pipeline de CI/CD.

Sugiro também que você crie um segredo para o seu usuário do DockerHub, que será usado para fazer o login no DockerHub. Para isso, siga os passos abaixo:

1. Acesse o repositório do GitHub onde você configurou o GitHub Actions.
2. Clique na aba "Settings" (Configurações) do repositório.
3. No menu lateral, clique em "Secrets and variables" e depois em "Actions".
4. Clique no botão "New repository secret" (Novo segredo do repositório).
5. Dê um nome para o segredo, como `DOCKERHUB_USERNAME`, e cole o seu nome de usuário do DockerHub.
6. Clique em "Add secret" (Adicionar segredo) para salvar o token.
7. Agora, o token estará disponível como um segredo no GitHub Actions e poderá ser usado no pipeline de CI/CD.

Com isso, você já tem os segredos necessários para fazer o upload da imagem Docker para o DockerHub. Agora, vamos configurar o GitHub Actions para fazer isso automaticamente sempre que houver um push no repositório.

### Configurando o GitHub Actions para fazer o upload da imagem Docker

Agora que temos os segredos armazenados no GitHub, precisamos configurar o GitHub Actions para fazer o upload da imagem Docker para o DockerHub. Para isso, vamos editar o arquivo `.github/workflows/ci.yml` que criamos anteriormente e adicionar as etapas necessárias para fazer o upload da imagem.

Aqui está um exemplo de como o arquivo `.github/workflows/ci.yml` pode ficar após as alterações:

```yaml title=".github/workflows/ci.yml" linenums="1" hl_lines="20-24 30"
name: ci-python
on: [push]
jobs:
  check-application:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.13'
          cache: 'pip'
      - run: pip install -r requirements.txt
      - run: pytest
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }} # (1)
          password: ${{ secrets.DOCKERHUB_TOKEN }} # (2)

      - name: Build and Push to DockerHub
        id: docker_build_push
        uses: docker/build-push-action@v2
        with:
          push: true # (3)
          tags: your-dockerhub-username/ci-sum-python:${{ github.sha }} # (4)
```

1. A expressão `${{ secrets.DOCKERHUB_USERNAME }}` indica o segredo que você criou para o seu nome de usuário do DockerHub.
2. A expressão `${{ secrets.DOCKERHUB_TOKEN }}` indica o segredo que você criou para o token de acesso do DockerHub.
3. A linha `push: true` indica que a imagem deve ser enviada para o DockerHub após a construção.
4. Aqui você deve substituir `your-dockerhub-username` pelo seu nome de usuário do DockerHub, como configurado nos segredos, usando `${{ secrets.DOCKERHUB_USERNAME }}`. O `ci-sum-python` é o nome da imagem que você deseja usar no DockerHub, e `${{ github.sha }}` é a tag da imagem que será gerada.

Note que usamos a variável `${{ github.sha }}` para gerar uma tag única para cada commit, o que é útil para identificar a versão da imagem correspondente ao commit. Você pode personalizar essa tag de acordo com suas necessidades, como usar um número de versão específico ou uma tag mais amigável. Também pode ser usada a palavra `latest`, mas somente isso não é recomendado, pois pode causar confusão ao identificar qual versão da imagem está sendo usada.

### Executando o GitHub Actions

Para executar o GitHub Actions, faça um push das alterações no arquivo `.github/workflows/ci.yml` para o repositório remoto. Após o push, o GitHub Actions será acionado automaticamente e você poderá visualizar o status da execução na aba "Actions" do repositório.

Se tudo estiver configurado corretamente, o GitHub Actions irá construir a imagem Docker e fazer o upload para o DockerHub automaticamente. Você pode verificar se a imagem foi enviada com sucesso acessando sua conta no DockerHub e verificando se a imagem `ci-sum-python` está disponível no seu repositório.

Em seguida, você pode usar a imagem Docker em qualquer lugar, como em um servidor de produção ou em um ambiente de desenvolvimento local, usando o comando `docker pull your-dockerhub-username/ci-sum-python:latest` ou `docker pull your-dockerhub-username/ci-sum-python:sha`, onde `sha` é o hash do commit que você deseja usar. Note que você deve estar atento ao nome que você deu para a imagem, na opção `tags`, e usar o mesmo nome para fazer o pull da imagem.
