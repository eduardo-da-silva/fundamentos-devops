# Orquestração de contêineres

Nos capítulos anteriores, utilizamos o Docker para construir imagens, gerenciar volumes, compor múltiplos serviços com o Docker Compose e preparar imagens com boas práticas para produção. Com o Compose, conseguimos definir e levantar ambientes com vários contêineres de forma declarativa e reproduzível. Isso funciona bem em um único servidor — mas o que acontece quando a aplicação precisa rodar em múltiplas máquinas, escalar sob demanda, ou se recuperar automaticamente de falhas?

O Docker Compose não foi projetado para resolver esses problemas. Ele não distribui contêineres entre máquinas, não reinicia serviços automaticamente em caso de falha de hardware, e não gerencia balanceamento de carga entre réplicas. É aí que entra a orquestração de contêineres.

## O que é orquestração

Orquestrar contêineres significa automatizar o gerenciamento do ciclo de vida de contêineres em ambientes distribuídos: provisionamento, escalonamento, monitoramento, rede, recuperação de falhas e atualização de versões. Em vez de gerenciar cada contêiner individualmente, você declara o estado desejado — por exemplo, "quero 3 réplicas desse serviço" — e a ferramenta de orquestração se encarrega de manter esse estado.

As responsabilidades típicas de um orquestrador incluem:

- **Escalonamento**: ajustar o número de réplicas de um serviço com base na carga de trabalho.
- **Balanceamento de carga**: distribuir o tráfego entre as réplicas disponíveis.
- **Recuperação automática**: reiniciar ou substituir contêineres que falharam.
- **Rede**: gerenciar a comunicação entre contêineres, mesmo em máquinas diferentes.
- **Gerenciamento de configuração e segredos**: centralizar variáveis de ambiente, senhas e chaves.
- **Atualizações graduais**: implantar novas versões sem interromper o serviço (rolling updates).

## Kubernetes

O Kubernetes (frequentemente abreviado como K8s) é a ferramenta de orquestração de contêineres mais adotada na indústria. Foi desenvolvido inicialmente pelo Google, baseado em mais de uma década de experiência interna com gerenciamento de contêineres (projeto Borg), e liberado como código aberto em 2014. Hoje é mantido pela Cloud Native Computing Foundation (CNCF).

Existem outras ferramentas de orquestração — Docker Swarm, HashiCorp Nomad, Apache Mesos — mas o Kubernetes se consolidou como o padrão de mercado. A maioria dos provedores de nuvem oferece serviços gerenciados de Kubernetes (EKS na AWS, GKE no Google Cloud, AKS na Azure).

### Arquitetura de um cluster

Um cluster Kubernetes é composto por dois tipos de nós:

- **Control plane** (plano de controle): responsável por gerenciar o cluster. Contém o servidor de API (`kube-apiserver`), o escalonador (`kube-scheduler`), o gerenciador de controladores (`kube-controller-manager`) e o banco de dados de estado (`etcd`). É nele que você envia comandos e declarações de recursos.

- **Worker nodes** (nós de trabalho): onde os contêineres efetivamente executam. Cada worker node roda o `kubelet` (agente que se comunica com o control plane) e o `kube-proxy` (gerenciamento de rede).

Em ambientes de produção, o control plane e os workers são máquinas separadas (físicas ou virtuais). Para desenvolvimento e estudo, podemos simular tudo em uma única máquina.

### Recursos fundamentais

Antes de colocar a mão na prática, é importante conhecer os recursos (objetos) que o Kubernetes gerencia. Veremos cada um em detalhes nas próximas páginas, mas segue uma visão inicial:

| Recurso        | Descrição                                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------------------------- |
| **Pod**        | Menor unidade de implantação. Contém um ou mais contêineres que compartilham rede e armazenamento.         |
| **Deployment** | Gerencia a criação e atualização de pods. Garante que o número desejado de réplicas esteja sempre rodando. |
| **Service**    | Expõe um conjunto de pods como um serviço de rede, com IP estável e DNS interno.                           |
| **ConfigMap**  | Armazena configurações não sensíveis (variáveis de ambiente, arquivos de configuração).                    |
| **Secret**     | Armazena dados sensíveis (senhas, tokens, chaves).                                                         |

### kubectl

O `kubectl` é a ferramenta de linha de comando para interagir com clusters Kubernetes. Todos os comandos que executaremos nas aulas passam por ele. Alguns comandos essenciais:

```bash
kubectl get nodes          # lista os nós do cluster
kubectl get pods           # lista os pods em execução
kubectl get deployments    # lista os deployments
kubectl get services       # lista os services
kubectl describe pod NOME  # detalhes de um pod específico
kubectl logs NOME          # logs de um pod
kubectl apply -f ARQ.yaml  # aplica uma configuração declarativa
kubectl delete -f ARQ.yaml # remove os recursos declarados no arquivo
```

## Ambiente local com Kind

Para as aulas, faremos uso do Kind (Kubernetes IN Docker), que permite criar clusters Kubernetes locais usando contêineres Docker como nós. É uma alternativa leve para o Minikube e não exige máquinas virtuais.

### Instalação

Pré-requisito: Docker instalado e funcionando.

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.28.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Instale também o `kubectl`, caso ainda não tenha:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

### Criando um cluster simples

```bash
kind create cluster
```

Verifique se o cluster está ativo:

```bash
kubectl cluster-info --context kind-kind
```

Você pode criar clusters com nomes específicos:

```bash
kind create cluster --name devops
```

E listar ou excluir:

```bash
kind get clusters
kind delete cluster --name devops
```

### Criando um cluster multi-nó

Para simular um ambiente mais próximo da produção — com um control plane e múltiplos workers — crie um diretório para os arquivos do projeto e dentro dele um arquivo de configuração:

```yaml title="kind.yaml" linenums="1"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

Aplique a configuração:

```bash
kind create cluster --config kind.yaml --name devops
```

Verifique os nós do cluster:

```bash
kubectl get nodes
```

Você deve ver 4 nós: 1 control-plane e 3 workers. Esse será o cluster que utilizaremos para os exercícios das próximas aulas.

## Carregando imagens locais no Kind

Um detalhe importante: como o Kind roda o Kubernetes dentro de contêineres Docker, ele não tem acesso direto às imagens que você construiu localmente. Para usar uma imagem local no cluster Kind sem publicá-la no DockerHub, use:

```bash
kind load docker-image seu-usuario/soma-api --name devops
```

Isso copia a imagem para dentro dos nós do cluster. Alternativa: publicar a imagem no DockerHub (como fizemos no capítulo anterior) e referenciá-la diretamente nos manifests.

## Exercícios

1. Instale o Kind e o kubectl na sua máquina. Crie um cluster simples (sem arquivo de configuração) e execute `kubectl get nodes`. Quantos nós aparecem? Qual o papel (role) dele?

2. Crie um cluster multi-nó com 1 control-plane e 3 workers usando o arquivo `kind.yaml`. Execute `kubectl get nodes -o wide` e observe os endereços IP de cada nó.

3. Execute `kubectl get namespaces` e anote os namespaces padrão que já existem. Pesquise para que serve cada um deles (`default`, `kube-system`, `kube-public`, `kube-node-lease`).

4. Carregue a imagem `seu-usuario/soma-api` (construída no capítulo anterior) no cluster com `kind load docker-image`. Verifique que a imagem está disponível executando `docker exec -it devops-worker crictl images` (substitua `devops-worker` pelo nome real do nó).
