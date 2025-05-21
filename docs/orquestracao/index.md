# Orquestração de contêineres

A orquestração de containers automatiza o gerenciamento de containers, desde o provisionamento até o escalonamento e monitoramento, permitindo que as organizações implantem aplicações em grande escala de forma mais eficiente. A orquestração é especialmente útil em ambientes de microserviços, onde aplicações são divididas em pequenos serviços independentes que podem ser implantados e escalonados separadamente. Além disso, ela ajuda a manter a coesão e a resiliência das aplicações, garantindo que os serviços estejam sempre disponíveis e funcionando corretamente.

Quando levantamos um container, usando o Docker por exemplo, estamos criando uma instância isolada de um aplicativo ou serviço. No entanto, à medida que o número de containers aumenta, torna-se difícil gerenciá-los manualmente. A orquestração de containers resolve esse problema, permitindo que os desenvolvedores e operadores automatizem tarefas repetitivas e complexas.

A orquestração de containers é uma abordagem que permite gerenciar e automatizar o ciclo de vida de containers em ambientes de produção. Isso inclui tarefas como provisionamento, escalonamento, monitoramento e recuperação de falhas. Ferramentas de orquestração ajudam a garantir que os containers estejam sempre disponíveis, escaláveis e funcionando corretamente.

Algumas características comuns de ferramentas de orquestração de containers incluem:

- **Gerenciamento de ciclo de vida**: Automatiza o provisionamento, escalonamento e monitoramento de containers.
- **Escalonamento automático**: Ajusta automaticamente o número de containers em execução com base na carga de trabalho.
- **Balanceamento de carga**: Distribui o tráfego entre os containers para garantir desempenho e disponibilidade.
- **Recuperação automática**: Reinicia containers com falha ou substitui containers não saudáveis.
- **Gerenciamento de configuração**: Permite a configuração centralizada de containers, facilitando a atualização e o gerenciamento de aplicações.
- **Rede e armazenamento**: Facilita a comunicação entre containers e o gerenciamento de volumes de armazenamento.
- **Segurança**: Fornece recursos de segurança, como autenticação, autorização e criptografia de dados.
- **Monitoramento e registro**: Coleta métricas e logs dos containers para análise e solução de problemas.
- **Integração com CI/CD**: Facilita a integração com pipelines de entrega contínua e implantação contínua.
- **Multi-cloud e híbrido**: Suporte para implantações em ambientes de nuvem pública, privada e híbrida.
- **Flexibilidade**: A capacidade de personalizar e estender a ferramenta para atender às necessidades específicas da organização.
- **Gerenciamento de segredos**: Permite armazenar e gerenciar segredos, como senhas e chaves de API, de forma segura.

Nas nossas aulas, faremos uso da ferramenta "kind" para facilitar a criação e gerenciamento de clusters Kubernetes locais. O Kind (Kubernetes IN Docker) é uma ferramenta que permite executar clusters Kubernetes em containers Docker, tornando mais fácil o desenvolvimento e teste de aplicações em ambientes Kubernetes. Ele é especialmente útil para desenvolvedores que desejam experimentar o Kubernetes sem precisar configurar um cluster completo em uma máquina física ou virtual. Por isso, é amplamente utilizado em ambientes de desenvolvimento e teste, onde a criação rápida e fácil de clusters Kubernetes é essencial.

## Instalação do Kind e comandos básicos

Primeiramente, para instalar o Kind é necessário ter o Docker instalado na máquina. Para isso, caso você ainda não tenha o Docker, pode consultar a aula referente aos contêineres.

Com o Docker instalado, você pode instalar o Kind usando o comando abaixo, em um ambiente Linux:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.28.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Depois, você pode criar um _cluster Kubernetes_ usando o Kind com o seguinte comando:

```bash
kind create cluster
```

Isso irá criar um cluster Kubernetes local em sua máquina usando containers Docker. Você pode verificar se o cluster foi criado com sucesso executando o seguinte comando:

```bash
kubectl cluster-info --context kind-kind
```

Se tudo estiver funcionando corretamente, você verá informações sobre o cluster Kubernetes em execução. O comando `kubectl` é a ferramenta de linha de comando do Kubernetes, que permite interagir com o cluster e gerenciar recursos. A instalação do `kubectl` é feita automaticamente quando você instala o Kind, portanto, não é necessário instalá-lo separadamente.

Também, é possível informar o nome do _cluster_ com o seguinte comando:

```bash
kind create cluster --name devops
```

Nesse caso, foi criado um cluster chamado "devops". Agora, você pode listar os cluster com o seguinte comando:

```bash
kind get clusters
```

O comando acima irá listar todos os clusters criados com o Kind e, nosso exemplo, deve retornar o seguinte resultado:

```bash
kind
devops
```

O primeiro cluster criado com o Kind é chamado "kind". O segundo cluster que criamos é chamado "devops". Você pode usar esses nomes para gerenciar os clusters individualmente, como iniciar, parar ou excluir um cluster específico.

Para excluir um cluster, você pode usar o seguinte comando:

```bash
kind delete cluster --name devops
```

Isso irá remover o cluster "devops" que criamos anteriormente. Se você quiser excluir o cluster padrão chamado "kind", basta executar:

```bash
kind delete cluster
```

Isso irá remover o cluster "kind" que foi criado automaticamente quando você instalou o Kind. Lembre-se de que, ao excluir um cluster, todos os dados e configurações associados a ele serão perdidos. Portanto, certifique-se de fazer backup de qualquer dado importante antes de excluir um cluster.

## Criando um cluster com o Kind

Para as demais atividades com o kubernetes que desenvolveremos, sugiro que você crie um diretório vazio e armazene os arquivos de configuração do cluster nesse diretório. Dentro desse diretório, criaremos um arquivo chamado `kind.yaml` com a configuração do cluster. O conteúdo desse arquivo é o seguinte:

```yaml title="./kind.yaml" linenums="1"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

Nesse arquivo de configuração, estamos criando um cluster com um nó de controle (control-plane) e três nós de trabalho (worker). O nó de controle é responsável por gerenciar o cluster, enquanto os nós de trabalho executam as aplicações e serviços. Note que esse é um arquivo declarativo, ou seja, ele descreve o estado desejado do cluster. O Kind irá criar o cluster com base nessa configuração. Vamos explicar as partes do arquivo:

- `kind: Cluster`: Especifica que estamos criando um cluster Kubernetes.
- `apiVersion: kind.x-k8s.io/v1alpha4`: Especifica a versão da API do Kind que estamos usando.
- `nodes`: Define os nós do cluster. Cada nó pode ter um papel diferente, como control-plane ou worker.

Para aplicar essa configuração e criar o cluster, execute o seguinte comando no terminal:

```bash
kind create cluster --config kind.yaml --name devops
```

Isso irá criar um cluster Kubernetes com a configuração especificada no arquivo `kind.yaml`. Após a criação do cluster, você pode verificar se ele está em execução usando o comando:

```bash
kubectl cluster-info --context kind-devops
```

## Criando uma aplicação

Vamos agora criar uma aplicação que será utilizada em todo o percurso de estudo do kubernetes. Como ela é uma aplicação apenas para fins de demonstração, será desenvolvida da forma mais simples possível. Para isso, faremos uso do framework FastApi, que é um framework web para construção de APIs em Python.

Inicialmente, crie e acesse um diretório de nome `app`:

```bash
mkdir app
cd app
```

Agora, crie um ambiente virtual `venv` e ative esse ambiente:

```bash
python -m venv .venv
source .venv/bin/activate
```

Em seguida, crie um arquivo `main.py` com o seguinte conteúdo:

```python title="./app/main.py" linenums="1"
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World!"}

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

Para testarmos a aplicação, podemos executar o seguinte comando:

```bash
python main.py
```

Isso irá iniciar o servidor da aplicação FastAPI. Você pode acessar a aplicação no navegador ou usar uma ferramenta como o `curl` para fazer uma requisição HTTP GET para o endpoint `/`, ou acessar http://localhost:8000. O servidor irá responder com a mensagem:

```json
{ "message": "Hello World!" }
```

Com o projeto FastAPI já desenvolvido, podemos desativar o ambiente virtual e voltar para o diretório de trabalho:

```bash
deactivate
cd ..
```

## Criando a imagem Docker

Com a aplicação desenvolvida, vamos criar uma imagem docker para ser publicada e depois provisionada e implantada num cluster k8s. Para tal, crie um arquivo `Dockerfile` com o seguinte conteúdo:

```docker title="./Dockerfile" linenums="1"
FROM python:3.13-slim
WORKDIR /app
COPY app/requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY app/app.py app.py
CMD ["python", "app.py"]
```

O arquivo declara uma imagem simples para a aplicação que foi desenvolvida. Abaixo, um detalhamento de cada linha:

- `FROM python:3.13-slim`: Especifica a imagem base a ser utilizada, que é uma versão leve do Python 3.13.
- `WORKDIR /app`: Define o diretório de trabalho dentro do contêiner como `/app`.
- `COPY app/requirements.txt requirements.txt`: Copia o arquivo `requirements.txt` para o diretório de trabalho no contêiner.
- `RUN pip install --no-cache-dir -r requirements.txt`: Instala as dependências da aplicação especificadas no `requirements.txt`.
- `COPY app/app.py app.py`: Copia o arquivo `app.py` para o diretório de trabalho no contêiner.
- `CMD ["python", "app.py"]`: Define o comando a ser executado quando o contêiner for iniciado, que é rodar a aplicação FastAPI.

Para testar, vamos construir a imagem. Aqui, usarei o meu nome de usuário do DockerHub (`eduardosilvasc`). Você deve usar o seu usuário para depois fazer a publicação da imagem:

```bash
docker build -t eduardosilvasc/hello-fastapi-k8s .
```

Para testar se a imagem foi criada corretamente, execute o seguinte comando:

```bash
docker run -p 8000:8000 eduardosilvasc/hello-fastapi-k8s
```

Com isso, o servidor da aplicação FastAPI estará rodando no contêiner e você poderá acessá-lo em http://localhost:8000. A resposta deve ser a mesma que obtivemos anteriormente:

```json
{ "message": "Hello World!" }
```

Agora, para publicar a imagem no DockerHub, execute o seguinte comando:

```bash
docker push eduardosilvasc/hello-fastapi-k8s
```

Claro que para isso, você deve estar logado no DockerHub. Caso não tenha feito isso, execute o seguinte comando:

```bash
docker login
```

Lembre que você precisará pegar as credenciais nas configurações do DockerHub.

## Criando um pod

Nessa etapa, vamos criar um pod que utiliza a imagem que criamos e registramos no repositório de imagens do DockerHub. Para isso, crie um arquivo chamado `pod.yaml`, no diretório `k8s`, com o seguinte conteúdo:

```yaml title="./k8s/pod.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: hello-fastapi
spec:
  containers:
    - name: hello-fastapi
      image: eduardosilvasc/hello-fastapi-k8s
      ports:
        - containerPort: 8000
```

Com o arquivo acima, estamos declarando um pod Kubernetes chamado `hello-fastapi`, que executa um contêiner baseado na imagem `eduardosilvasc/hello-fastapi-k8s` e expõe a porta 8000. Para criar o pod, execute o seguinte comando:

```bash
kubectl apply -f k8s/pod.yaml
```

Para verificar se o pod foi criado e está em execução, use o comando:

```bash
kubectl get pods
```

Você deve ver o pod `hello-fastapi` na lista de pods em execução. Para acessar a aplicação FastAPI que está rodando no pod, você pode usar o port-forwarding do Kubernetes:

```bash
kubectl port-forward pod/hello-fastapi 8000:8000
```

Agora, você pode acessar a aplicação em http://localhost:8000. A resposta deve ser a mesma que obtivemos anteriormente:

```json
{ "message": "Hello World!" }
```

## Criando um Deployment

A criação de um pod é a parte essencial de um cluster k8s. Contudo, da forma que criamos esse recurso, o sistema não está preparado para lidar com falhas ou escalabilidade. Por exemplo, se o pod falhar ou for excluído, ele não será reiniciado automaticamente. Para resolver isso, podemos usar um recurso chamado Deployment, que é uma abstração de nível superior que gerencia a criação e atualização de pods.

Para isso, vamos criar um arquivo chamado `deployment.yaml`, no diretório `k8s`, com o seguinte conteúdo:

```yaml title="./k8s/deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-fastapi
spec:
    replicas: 3
    selector:
        matchLabels:
        app: hello-fastapi
    template:
        metadata:
        labels:
            app: hello-fastapi
        spec:
        containers:
        - name: hello-fastapi
            image: eduardosilvasc/hello-fastapi-k8s
            ports:
            - containerPort: 8000
```

Na configuração acima do Deployment, estamos especificando que queremos 3 réplicas do pod `hello-fastapi`. O Kubernetes irá garantir que sempre haja 3 pods em execução. Se um pod falhar ou for excluído, o Kubernetes irá criar um novo pod automaticamente para substituí-lo.
Para criar o Deployment, execute o seguinte comando:

```bash
kubectl apply -f k8s/deployment.yaml
```

Para verificar se o Deployment foi criado e está em execução, use o comando:

```bash
kubectl get deployments
```

Você deve ver o Deployment `hello-fastapi` na lista de deployments em execução. Para verificar os pods criados pelo Deployment, use o comando:

```bash
kubectl get pods
```

Você deve ver 3 pods `hello-fastapi` em execução. Para acessar a aplicação FastAPI que está rodando nos pods, você pode usar o port-forwarding do Kubernetes:

```bash
kubectl port-forward deployment/hello-fastapi 8000:8000
```

Agora, você pode acessar a aplicação em http://localhost:8000. A resposta deve ser a mesma que obtivemos anteriormente:

```json
{ "message": "Hello World!" }
```

## Criando um Service

Para expor a aplicação FastAPI para o mundo externo, precisamos criar um Service. Um Service é um recurso do Kubernetes que define como acessar os pods em execução. Ele fornece um ponto de acesso estável para os pods, mesmo que os pods sejam criados ou excluídos.
Para isso, vamos criar um arquivo chamado `service.yaml`, no diretório `k8s`, com o seguinte conteúdo:

```yaml title="./k8s/service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: hello-fastapi-service
spec:
  selector:
    app: hello-fastapi
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
```

Nesse arquivo, estamos criando um Service chamado `hello-fastapi` que seleciona os pods com o rótulo `app: hello-fastapi`. O Service irá expor a porta 80 e encaminhar o tráfego para a porta 8000 dos pods. Para criar o Service, execute o seguinte comando:

```bash
kubectl apply -f k8s/service.yaml
```

Para verificar se o Service foi criado e está em execução, use o comando:

```bash
kubectl get services
```

Você deve ver o Service `hello-fastapi` na lista de serviços em execução. Para acessar a aplicação FastAPI que está rodando nos pods, você pode usar o port-forwarding do Kubernetes:

```bash
kubectl port-forward service/hello-fastapi-service 8000:80
```

Agora, você pode acessar a aplicação em http://localhost:8000. A resposta deve ser a mesma que obtivemos anteriormente:

```json
{ "message": "Hello World!" }
```
