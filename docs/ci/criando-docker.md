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
