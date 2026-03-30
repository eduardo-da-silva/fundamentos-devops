# Boas práticas e imagens de produção

Nos capítulos anteriores, aprendemos a construir imagens Docker, persistir dados e orquestrar múltiplos contêineres com o Docker Compose. Antes de avançarmos para o Kubernetes, é importante consolidar algumas práticas que tornam as imagens mais seguras, menores e adequadas para ambientes de produção.

Neste capítulo, também vamos evoluir o projeto que construímos na integração contínua — aquele script Python com testes — transformando-o em uma API web com FastAPI. O objetivo é manter um projeto único ao longo da disciplina, que cresce a cada etapa.

## Evolução do projeto: de script para API

Na integração contínua, criamos um projeto com uma função `sumF` e testes com pytest. Agora, vamos transformá-lo em uma API HTTP usando o FastAPI. Essa evolução é natural: uma API pode ser testada, containerizada e depois implantada em um cluster Kubernetes.

Partindo do mesmo repositório que já possui `app.py` e `test_app.py`, vamos reestruturar o projeto.

### Instalando as dependências

Ative o ambiente virtual do projeto e instale o FastAPI e o Uvicorn:

```bash
source .venv/bin/activate
pip install fastapi uvicorn
```

### Reestruturando o código

Substitua o conteúdo do arquivo `app.py` pelo seguinte:

```python title="app.py" linenums="1"
from fastapi import FastAPI

app = FastAPI()


def sum_values(a: int, b: int) -> int:
    return a + b


@app.get("/")
def read_root():
    return {"message": "API de soma funcionando"}


@app.get("/sum/{a}/{b}")
def get_sum(a: int, b: int):
    return {"result": sum_values(a, b)}


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

A função de soma continua existindo — agora ela é acessível via endpoint HTTP. Atualize também o arquivo de testes:

```python title="test_app.py" linenums="1"
from fastapi.testclient import TestClient
from app import app, sum_values


def test_sum_values():
    assert sum_values(1, 2) == 3
    assert sum_values(2, 2) == 4
    assert sum_values(3, 2) == 5


def test_root_endpoint():
    client = TestClient(app)
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "API de soma funcionando"}


def test_sum_endpoint():
    client = TestClient(app)
    response = client.get("/sum/10/5")
    assert response.status_code == 200
    assert response.json() == {"result": 15}
```

Atualize o `requirements.txt`:

```bash
pip freeze > requirements.txt
```

Execute os testes para confirmar que tudo funciona:

```bash
pytest
```

Se os testes passarem, faça o commit e o push para o repositório.

## O arquivo .dockerignore

Assim como o `.gitignore` impede que arquivos indesejados entrem no repositório, o `.dockerignore` impede que eles sejam copiados para dentro da imagem Docker durante o `COPY` ou `ADD`.

Sem esse arquivo, o comando `COPY . .` copiaria tudo — incluindo o ambiente virtual, cache do pytest, diretório `.git`, entre outros. Isso aumenta o tamanho da imagem e pode expor informações desnecessárias.

Crie um arquivo `.dockerignore` na raiz do projeto:

```plaintext title=".dockerignore"
.venv
__pycache__
.pytest_cache
.git
.gitignore
*.pyc
```

## Multi-stage builds

Até aqui, usamos Dockerfiles simples com um único estágio (`FROM`). Isso funciona, mas tem uma limitação: tudo o que é necessário para a **construção** da aplicação acaba presente na imagem final — ferramentas de build, compiladores, cabeçalhos de desenvolvimento. Em projetos maiores, isso pode dobrar ou triplicar o tamanho da imagem.

O multi-stage build resolve isso separando a construção da execução. A ideia é usar um estágio para instalar dependências e preparar a aplicação, e outro estágio — com uma imagem mais enxuta — para copiar apenas o necessário e executar.

### Dockerfile com estágio único (antes)

```docker title="Dockerfile" linenums="1"
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### Dockerfile com multi-stage (depois)

```docker title="Dockerfile" linenums="1"
# Estágio de build: instala dependências
FROM python:3.13-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Estágio final: imagem de execução
FROM python:3.13-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app.py .
CMD ["python", "app.py"]
```

No primeiro estágio (`builder`), instalamos as dependências em um diretório separado (`/install`). No segundo estágio, copiamos apenas as bibliotecas instaladas e o código da aplicação. Ferramentas de build ou arquivos temporários do primeiro estágio não entram na imagem final.

Para verificar a diferença no tamanho das imagens, construa as duas versões e compare:

```bash
docker build -t soma-api:single -f Dockerfile.single .
docker build -t soma-api:multi -f Dockerfile .
docker images | grep soma-api
```

Em projetos com dependências que exigem compilação (como pacotes com extensões C), a diferença é ainda mais significativa.

## HEALTHCHECK

Em ambientes de produção, não basta um contêiner estar "rodando" — é preciso saber se a aplicação dentro dele está respondendo corretamente. A instrução `HEALTHCHECK` no Dockerfile define um comando que o Docker executa periodicamente para verificar a saúde do contêiner.

Adicione ao final do Dockerfile, antes do `CMD`:

```docker title="Dockerfile" linenums="1"
FROM python:3.13-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.13-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app.py .

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/')" || exit 1

CMD ["python", "app.py"]
```

Os parâmetros significam:

- `--interval=30s`: executa a verificação a cada 30 segundos.
- `--timeout=5s`: considera falha se não responder em 5 segundos.
- `--start-period=10s`: tempo de espera antes da primeira verificação (para a aplicação iniciar).
- `--retries=3`: marca como _unhealthy_ após 3 falhas consecutivas.

Após construir e rodar o contêiner, você pode verificar o status de saúde com:

```bash
docker build -t soma-api .
docker run -d --name api-teste soma-api
docker inspect --format='{{.State.Health.Status}}' api-teste
```

O status será `starting`, `healthy` ou `unhealthy`. Ferramentas de orquestração como o Kubernetes utilizam mecanismos semelhantes (liveness e readiness probes) para decidir se um contêiner deve ser reiniciado ou removido do balanceamento de carga.

## Dockerfile final do projeto

Reunindo todas as práticas, o Dockerfile do projeto fica assim:

```docker title="Dockerfile" linenums="1"
FROM python:3.13-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.13-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY app.py .

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/')" || exit 1

CMD ["python", "app.py"]
```

Construa e teste:

```bash
docker build -t seu-usuario/soma-api .
docker run -d -p 8000:8000 --name api-teste seu-usuario/soma-api
```

Acesse http://localhost:8000 para verificar a raiz, e http://localhost:8000/sum/3/4 para testar o endpoint de soma. Após o teste, limpe o contêiner:

```bash
docker stop api-teste && docker rm api-teste
```

## Publicando a imagem atualizada

Faça login no DockerHub e publique:

```bash
docker login
docker push seu-usuario/soma-api
```

Essa é a imagem que utilizaremos nos próximos capítulos, quando implantarmos a aplicação em um cluster Kubernetes.

## Exercícios

1. Construa o Dockerfile do projeto em modo single-stage e multi-stage. Compare o tamanho das duas imagens com `docker images`. Qual a diferença? Por que ela existe?

2. Remova o `HEALTHCHECK` do Dockerfile, construa e rode o contêiner. Depois, pare a aplicação dentro do contêiner (sem parar o contêiner em si). O Docker percebe que a aplicação parou de responder? Agora adicione o `HEALTHCHECK` de volta e repita o experimento. Qual a diferença no comportamento?

3. Crie um `.dockerignore` vazio e construa a imagem com `COPY . .`. Depois, adicione as regras adequadas e reconstrua. Compare o tamanho e o tempo de build.

4. (Proposta aberta) Adicione um novo endpoint à API — por exemplo, `/subtract/{a}/{b}` — com o respectivo teste. Reconstrua a imagem e verifique que o novo endpoint funciona.
