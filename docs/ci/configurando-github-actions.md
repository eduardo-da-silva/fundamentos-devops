## Configuração do GitHub Actions

O GitHub Actions permite a criação de pipelines de CI/CD diretamente no repositório, facilitando a automação de tarefas como testes, compilação, análise de código e implantação. Para configurar o GitHub Actions, crie um arquivo chamado `.github/workflows/ci.yml` na raiz do projeto com o seguinte conteúdo:

```yaml
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

      - name: Build
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          push: false
```

Neste arquivo de configuração, definimos um job chamado `check-application` que executa as seguintes etapas:

1. `actions/checkout@v4`: Faz o checkout do código fonte do repositório.
2. `actions/setup-python@v5`: Configura o ambiente Python com a versão 3.13 e o cache do `pip`.
3. `pip install -r requirements.txt`: Instala as dependências do projeto.
4. `pytest`: Executa os testes unitários do projeto.
5. `docker/setup-qemu-action@v1`: Configura o QEMU para execução de imagens multiplataforma.
6. `docker/setup-buildx-action@v1`: Configura o Buildx para construção de imagens Docker.
7. `docker/build-push-action@v2`: Constrói a imagem Docker a partir do `Dockerfile`.

Note que o job `check-application` é acionado sempre que ocorre um push no repositório. Você pode personalizar as etapas de acordo com as necessidades do seu projeto, adicionando mais comandos, testes e ações específicas.

Também, na última etapa, foi configurada a opção `push` como `false`, o que significa que a imagem Docker será construída, mas não será enviada para um registro de contêiner. Você pode alterar essa opção para `true` e adicionar as credenciais necessárias para fazer o push da imagem para um registro de contêiner. Nesta etapa, estamos apenas construindo a imagem para fins de demonstração.

### Executando o GitHub Actions

Para executar o GitHub Actions, faça um push das alterações no arquivo `.github/workflows/ci.yml` para o repositório remoto. Após o push, o GitHub Actions será acionado automaticamente e você poderá visualizar o status da execução na aba "Actions" do repositório.
