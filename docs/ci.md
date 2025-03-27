# Integração Contínua

A integração contínua (CI) é uma prática de desenvolvimento de software que consiste em integrar o código dos desenvolvedores em um repositório compartilhado com frequência, de preferência várias vezes ao dia. O objetivo da integração contínua é detectar e corrigir problemas de integração o mais rápido possível, garantindo que o código seja sempre funcional e esteja em conformidade com as expectativas do projeto.

Na prática, a integração contínua envolve a automação de tarefas como compilação, testes e análise estática de código, que são executadas automaticamente sempre que um novo código é integrado no repositório. Isso permite que a equipe de desenvolvimento identifique e corrija problemas rapidamente, evitando que bugs e erros se acumulem no código.

Entre as principais funcionalidades da integração contínua, destacam-se:

- **Automação de testes:** a integração contínua permite a execução automática de testes unitários, testes de integração e testes de aceitação sempre que o código é integrado. Isso garante que o código seja testado continuamente e que novos bugs sejam identificados rapidamente.
- **Compilação automática:** a integração contínua também inclui a compilação automática do código fonte sempre que uma alteração é integrada. Isso permite que a equipe verifique se o código compila corretamente e se não há erros de compilação.
- **Linter e análise estática de código:** a integração contínua também pode incluir a execução de ferramentas de análise estática de código e linters para identificar possíveis problemas de código, como más práticas, código duplicado e vulnerabilidades de segurança.
- **Verificação da qualidade do código:** a integração contínua também pode incluir a verificação da qualidade do código através de métricas de complexidade, cobertura de testes e outras métricas de qualidade de código.
- **Verificação de segurança:** a integração contínua também pode incluir a verificação de vulnerabilidades de segurança no código fonte, garantindo que o código seja seguro e livre de vulnerabilidades conhecidas.
- **Geração de artefatos:** a integração contínua também pode incluir a geração de artefatos, como builds compiladas, pacotes de instalação e documentação, que são usados para implantação e distribuição do software.
- **Identificação de próximas versões:** a integração contínua pode ser usada para identificar próximas versões do software, gerando tags, versões e changelogs automaticamente.
- **Geração de tags e releases:** a integração contínua pode gerar tags e releases automaticamente, facilitando o versionamento e a distribuição do software.

Entre as principais ferramentas de integração contínua disponíveis no mercado, destacam-se o GitHub Actions, GitLab CI, Jenkins, Travis CI, CircleCI e Azure DevOps. Cada uma dessas ferramentas oferece recursos avançados de automação, integração com repositórios Git e suporte para diversas linguagens de programação e tecnologias.

Aqui, abordaremos apenas o GitHub Actions, que é uma ferramenta de automação de fluxos de trabalho integrada ao GitHub. O GitHub Actions permite a criação de pipelines de CI/CD diretamente no repositório, facilitando a automação de tarefas como testes, compilação, análise de código e implantação. Além disso, possui uma comunidade ativa que disponibiliza diversos workflows prontos para uso em diferentes cenários.

## Criando um projeto

Neste guia, vamos criar um projeto simples em Python para demonstrar a integração contínua com o GitHub Actions. O projeto consistirá em um script Python que realiza a soma de dois números e exibe o resultado.

### Passo a passo

- [x] Crie um diretório para o projeto.

Crie um diretório para o projeto em sua máquina local e abra-o com o Visual Studio Code ou outro editor de sua preferência.

- [x] Inicialize um repositório Git

Dentro do diretório do projeto, inicialize um repositório Git com o comando:

```bash
git init
```

- [x] Crie um ambiente virtual

Crie um ambiente virtual para o projeto com o comando:

```bash
python -m venv .venv
```

Em seguida, ative o ambiente virtual:

=== "Linux e macOS"

    ```bash
    source .venv/bin/activate
    ```

=== "Windows"

    ```powershell
    .venv\Scripts\activate
    ```

- [x] Configure o .gitignore

Crie um arquivo `.gitignore` na raiz do projeto e adicione o seguinte conteúdo para ignorar o ambiente virtual e os arquivos `.pyc`:

```plaintext title=".gitignore"
.venv
*.pyc
__pycache__/
.pytest_cache/
```

- [x] Instale o `pytest`

Instale o `pytest` para realizar os testes unitários do projeto:

```bash
pip install pytest
```

- [x] Crie um arquivo de testes

Crie um arquivo Python chamado `test_app.py` com o seguinte conteúdo:

```py title="test_app.py"
from app import sumF


def test_sum():
    assert sumF(1, 2) == 3
    assert sumF(2, 2) == 4
    assert sumF(3, 2) == 5

```

Se rodarmos os testes agora, eles devem falhar, pois ainda não implementamos a função `sumF`. Para testar, execute o comando:

```bash
pytest
```

- [x] Crie um arquivo Python

Crie um arquivo Python chamado `app.py` com o seguinte conteúdo:

```python title="app.py"
def sumF(a, b):
    return a + b


if __name__ == "__main__":
    print(sumF(10, 10))
```

- [x] Execute os testes

Execute os testes novamente com o comando `pytest`. Eles devem passar agora, pois a função `sumF` foi implementada e está correta.

- [x] Crie um arquivo `requirements.txt`

Crie um arquivo `requirements.txt` com as dependências do projeto:

```bash
pip freeze > requirements.txt
```

### Fazendo o commit inicial

Neste momento, você já pode publicar a branch principal do projeto no GitHub. Para isso, você pode usar a própria extensão do Visual Studio Code ou os comandos do Git.

Na próxima sessão, vamos configurar o GitHub Actions para realizar a integração contínua do projeto.

## Criação de uma imagem Docker

Para criar uma imagem Docker, é necessário criar um arquivo chamado `Dockerfile` na raiz do projeto. O `Dockerfile` é um arquivo de configuração que define como a imagem Docker será construída, incluindo as dependências, comandos e configurações necessárias para executar a aplicação.

Aqui está um exemplo de `Dockerfile` para um projeto Python simples:

```Dockerfile title="Dockerfile"
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY app.py app.py
CMD ["python", "app.py"]
```

No arquivo criado, temos as seguintes instruções:

- `FROM python:3.13-slim`: Define a imagem base a ser utilizada, neste caso, uma imagem Python 3.13 slim.
- `WORKDIR /app`: Define o diretório de trabalho dentro da imagem.
- `COPY requirements.txt requirements.txt`: Copia o arquivo `requirements.txt` para o diretório de trabalho.
- `RUN pip install -r requirements.txt`: Instala as dependências do projeto.
- `COPY app.py app.py`: Copia o arquivo `app.py` para o diretório de trabalho.
- `CMD ["python", "app.py"]`: Define o comando a ser executado quando o contêiner for iniciado.

Note que não rodamos o comando `pytest` dentro do contêiner, pois os testes unitários devem ser executados antes de construir a imagem Docker. Para isso, podemos configurar um job no GitHub Actions para executar os testes antes de construir a imagem.

### Testando a imagem Docker localmente

Para testar a imagem Docker localmente, você pode executar os seguintes comandos:

- [x] Construir a imagem:

```bash
docker build -t ci-sum-python .
```

- [x] Executar o contêiner:

```bash
docker run --rm ci-sum-python
```

O comando `docker build` cria uma imagem Docker com o nome `ci-sum-python` a partir do `Dockerfile` presente no diretório atual. Em seguida, o comando `docker run` inicia um contêiner a partir da imagem criada, executando o arquivo `app.py` e exibindo a saída no terminal.

No caso do nosso exemplo, o arquivo `app.py` simplesmente soma dois números e imprime o resultado. Ao executar o contêiner, você deve ver a saída correspondente à soma dos números.

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

```

```
