# Services e Rede

No capítulo anterior, implantamos a aplicação com um Deployment e acessamos via `kubectl port-forward`. O port-forward é útil para depuração, mas não é um mecanismo de rede real — ele cria um túnel temporário entre a sua máquina e um pod específico. Se o pod for recriado (o que o Deployment faz automaticamente), o port-forward perde a conexão.

Neste capítulo, veremos como o Kubernetes gerencia a rede interna do cluster e como expor aplicações de forma estável usando Services.

## Rede no Kubernetes

Cada pod no Kubernetes recebe um endereço IP próprio dentro do cluster. Pods podem se comunicar entre si usando esses IPs diretamente. No entanto, os IPs dos pods são efêmeros — quando um pod é recriado pelo Deployment, ele recebe um IP diferente. Isso torna inviável referenciar pods por IP.

O Kubernetes resolve esse problema com o recurso **Service**: um objeto que fornece um IP estável e um nome DNS para acessar um conjunto de pods. O Service seleciona os pods por labels (os mesmos labels definidos no Deployment) e distribui o tráfego entre as réplicas disponíveis.

## Tipos de Service

O Kubernetes oferece três tipos principais de Service:

| Tipo             | Descrição                                                            | Acesso                                    |
| ---------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| **ClusterIP**    | IP interno ao cluster. Padrão.                                       | Apenas de dentro do cluster.              |
| **NodePort**     | Expõe o serviço em uma porta fixa de cada nó do cluster.             | De fora do cluster, via IP do nó + porta. |
| **LoadBalancer** | Provisiona um balanceador de carga externo (em provedores de nuvem). | De fora do cluster, via IP público.       |

Em ambientes locais com Kind, o `LoadBalancer` não funciona nativamente (não há provedor de nuvem para provisionar o balanceador). Usaremos `ClusterIP` para comunicação interna e `NodePort` para acesso externo.

## Criando um Service do tipo ClusterIP

Vamos criar um Service que exponha internamente o Deployment da nossa API. Crie o arquivo `service.yaml` no diretório `k8s`:

```yaml title="k8s/service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: soma-api-service
spec:
  type: ClusterIP
  selector:
    app: soma-api
  ports:
    - port: 80
      targetPort: 8000
      protocol: TCP
```

Detalhamento:

- `selector.app: soma-api`: seleciona todos os pods que possuem o label `app: soma-api` — ou seja, os pods criados pelo nosso Deployment.
- `port: 80`: a porta em que o Service escuta dentro do cluster.
- `targetPort: 8000`: a porta do contêiner para onde o tráfego é encaminhado.

Aplique o manifest:

```bash
kubectl apply -f k8s/service.yaml
```

Verifique o Service criado:

```bash
kubectl get services
```

Você verá o `soma-api-service` com um IP de cluster (algo como `10.96.X.X`). Esse IP é estável — ele não muda mesmo que os pods sejam recriados.

### DNS interno

Além do IP, o Kubernetes cria automaticamente uma entrada de DNS para cada Service. O nome segue o formato:

```
<nome-do-service>.<namespace>.svc.cluster.local
```

No nosso caso: `soma-api-service.default.svc.cluster.local`. Dentro do mesmo namespace (`default`), basta usar `soma-api-service`.

Para testar, vamos criar um pod temporário e fazer uma requisição de dentro do cluster:

```bash
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- \
    curl http://soma-api-service/sum/10/5
```

A resposta deve ser `{"result":15}`. Isso confirma que o Service está funcionando e distribuindo o tráfego para os pods do Deployment.

## Criando um Service do tipo NodePort

O ClusterIP só é acessível de dentro do cluster. Para acessar a aplicação de fora (da sua máquina, por exemplo), podemos usar um NodePort:

```yaml title="k8s/service-nodeport.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: soma-api-nodeport
spec:
  type: NodePort
  selector:
    app: soma-api
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30080
      protocol: TCP
```

A diferença é o `type: NodePort` e o campo `nodePort: 30080`. Isso significa que a porta 30080 de cada nó do cluster será mapeada para o Service. O range de portas permitido para NodePort é 30000–32767.

```bash
kubectl apply -f k8s/service-nodeport.yaml
```

!!!note "Acesso ao NodePort com Kind"

    No Kind, os nós do cluster são contêineres Docker. Para acessar o NodePort, você precisa descobrir o IP do nó. Execute `kubectl get nodes -o wide` e use o endereço `INTERNAL-IP` de qualquer nó worker. Exemplo: `curl http://172.18.0.3:30080/sum/2/3`. Alternativamente, como o Kind roda em Docker, você pode usar `docker port` para verificar os mapeamentos de porta do contêiner do nó.

Na prática, mesmo com o NodePort, para acessar diretamente da máquina local no Kind, o port-forward ainda é a forma mais simples:

```bash
kubectl port-forward svc/soma-api-service 8000:80
```

Note que agora estamos fazendo port-forward para o **Service** (não para um pod individual). O tráfego é distribuído entre as réplicas.

## Exemplo prático: API + banco de dados

Uma das situações mais comuns em ambientes Kubernetes é ter uma aplicação que se comunica com um banco de dados. Vamos criar um deployment para o PostgreSQL e fazer a API acessá-lo via DNS interno.

### Deployment do PostgreSQL

```yaml title="k8s/postgres-deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: 'devops'
            - name: POSTGRES_PASSWORD
              value: 'devops123'
            - name: POSTGRES_DB
              value: 'soma_db'
```

### Service do PostgreSQL

```yaml title="k8s/postgres-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

Aplique ambos:

```bash
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
```

Agora, qualquer pod dentro do cluster pode acessar o PostgreSQL usando o endereço `postgres-service:5432`. O DNS interno resolve automaticamente para o IP do Service.

Para verificar a conectividade, crie um pod temporário com o cliente PostgreSQL:

```bash
kubectl run pg-test --image=postgres:16 --rm -it --restart=Never -- \
    psql -h postgres-service -U devops -d soma_db -c "SELECT 1;"
```

Se solicitada a senha, informe `devops123`. A consulta deve retornar o resultado, confirmando que a comunicação entre pods funciona via Service.

!!!warning "Sobre credenciais em variáveis de texto"

    No exemplo acima, as credenciais do banco estão diretamente no manifest como texto. Isso foi feito para simplificar a demonstração. Em ambientes reais, esses valores devem ser armazenados em **Secrets** — o que veremos no próximo capítulo.

### Visualizando a comunicação

Uma forma de visualizar toda a topologia de recursos é listar tudo de uma vez:

```bash
kubectl get all
```

Esse comando mostra pods, services, deployments e replicasets em uma única saída.

## Organização dos arquivos

Após este capítulo, o diretório `k8s/` contém:

```
k8s/
├── pod.yaml
├── deployment.yaml
├── service.yaml
├── service-nodeport.yaml
├── postgres-deployment.yaml
└── postgres-service.yaml
```

## Exercícios

1. Crie o Service do tipo ClusterIP para a API e teste o acesso usando um pod temporário com `curl` (como mostrado acima). Confirme que o tráfego está sendo distribuído: execute a requisição várias vezes e observe nos logs dos pods qual deles recebeu cada requisição (`kubectl logs <pod>`).

2. Crie um Service do tipo NodePort e acesse a API de fora do cluster. Anote o IP do nó e a porta utilizada.

3. Exclua um dos pods da API enquanto faz requisições ao Service. O Service percebe a exclusão e redireciona o tráfego para os pods restantes?

4. (Proposta aberta) Implante o PostgreSQL conforme mostrado acima. Crie um segundo Deployment com o Adminer (imagem `adminer`) e exponha-o via NodePort ou port-forward. Acesse o Adminer no navegador e conecte-se ao PostgreSQL usando o DNS interno (`postgres-service`). Esse exercício reforça a comunicação entre serviços dentro do cluster — o mesmo conceito que praticamos com Docker Compose, agora em Kubernetes.
