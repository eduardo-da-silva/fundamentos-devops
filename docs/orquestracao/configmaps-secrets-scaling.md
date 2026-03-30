# Configuração e Escalabilidade

Nos capítulos anteriores, criamos Deployments e Services com as configurações escritas diretamente nos manifests YAML — incluindo, no caso do PostgreSQL, credenciais em texto dentro do arquivo. Em aplicações reais, isso é um problema: o mesmo container pode precisar rodar com configurações diferentes em cada ambiente (desenvolvimento, homologação, produção), e senhas não devem estar versionadas no repositório.

O Kubernetes oferece dois recursos para separar a configuração do código: **ConfigMaps** e **Secrets**. Além disso, veremos o conceito de escalonamento automático com o **Horizontal Pod Autoscaler**.

## ConfigMap

O ConfigMap armazena dados de configuração não sensíveis na forma de pares chave-valor. Esses dados podem ser injetados nos pods como variáveis de ambiente ou como arquivos montados em um volume.

### Criando um ConfigMap

Vamos criar um ConfigMap para a nossa API, armazenando o nome da aplicação e a versão — valores que a aplicação poderia consumir como variáveis de ambiente.

```yaml title="k8s/configmap.yaml" linenums="1"
apiVersion: v1
kind: ConfigMap
metadata:
  name: soma-api-config
data:
  APP_NAME: 'API de Soma'
  APP_VERSION: '1.0.0'
  LOG_LEVEL: 'info'
```

Aplique:

```bash
kubectl apply -f k8s/configmap.yaml
```

Verifique o conteúdo:

```bash
kubectl describe configmap soma-api-config
```

### Usando o ConfigMap no Deployment

Para que os pods da API tenham acesso a essas variáveis, precisamos atualizar o Deployment. Edite o arquivo `k8s/deployment.yaml` e adicione a referência ao ConfigMap:

```yaml title="k8s/deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: soma-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: soma-api
  template:
    metadata:
      labels:
        app: soma-api
    spec:
      containers:
        - name: soma-api
          image: seu-usuario/soma-api
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef:
                name: soma-api-config
```

O campo `envFrom` com `configMapRef` injeta todas as chaves do ConfigMap como variáveis de ambiente no contêiner. Após aplicar, você pode verificar que as variáveis existem dentro do pod:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl exec -it $(kubectl get pods -l app=soma-api -o jsonpath='{.items[0].metadata.name}') -- env | grep APP
```

A saída deve mostrar `APP_NAME=API de Soma` e `APP_VERSION=1.0.0`.

### Variáveis individuais

Se quiser injetar apenas algumas chaves (e não todas), use `env` com `valueFrom`:

```yaml
env:
  - name: APP_NAME
    valueFrom:
      configMapKeyRef:
        name: soma-api-config
        key: APP_NAME
```

Essa abordagem dá mais controle sobre quais variáveis ficam visíveis no pod.

## Secret

O Secret funciona de forma semelhante ao ConfigMap, mas é projetado para dados sensíveis: senhas, tokens, chaves de API. Os valores são armazenados codificados em base64 dentro do cluster.

!!!warning "Sobre a segurança dos Secrets"

    Codificação base64 não é criptografia. Qualquer pessoa com acesso ao cluster pode decodificar os valores. Para ambientes de produção, é recomendável habilitar a encriptação em repouso (encryption at rest) no etcd, ou usar soluções externas como HashiCorp Vault ou Sealed Secrets. Para fins de estudo, o Secret padrão do Kubernetes é suficiente.

### Criando um Secret

Vamos migrar as credenciais do PostgreSQL (que deixamos em texto no capítulo anterior) para um Secret:

```yaml title="k8s/postgres-secret.yaml" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
type: Opaque
stringData:
  POSTGRES_USER: devops
  POSTGRES_PASSWORD: devops123
  POSTGRES_DB: soma_db
```

O campo `stringData` aceita valores em texto puro (o Kubernetes converte automaticamente para base64 ao armazenar). Aplique:

```bash
kubectl apply -f k8s/postgres-secret.yaml
```

Verifique (os valores aparecerão em base64):

```bash
kubectl get secret postgres-credentials -o yaml
```

### Usando o Secret no Deployment do PostgreSQL

Atualize o Deployment do PostgreSQL para usar o Secret em vez de valores em texto:

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
          envFrom:
            - secretRef:
                name: postgres-credentials
```

Aplique:

```bash
kubectl apply -f k8s/postgres-deployment.yaml
```

O comportamento é idêntico ao que tínhamos antes — mas agora as credenciais não estão no manifest do Deployment. Elas ficam no Secret, que pode ser gerenciado separadamente e não precisa ser versionado junto com o código da aplicação.

## Escalabilidade

### Escalonamento manual

Já vimos no capítulo de Deployments que podemos alterar o número de réplicas com `kubectl scale`:

```bash
kubectl scale deployment soma-api --replicas=5
```

Isso é útil quando sabemos antecipadamente qual carga o sistema receberá. Mas em muitos cenários, a carga varia ao longo do dia — e não queremos manter recursos ociosos nem ficar sem capacidade nos horários de pico.

### Horizontal Pod Autoscaler (HPA)

O Horizontal Pod Autoscaler ajusta automaticamente o número de réplicas de um Deployment com base em métricas observadas — tipicamente uso de CPU ou memória.

Para que o HPA funcione, precisamos de duas coisas:

1. **Definir limites de recursos nos pods** (para que o Kubernetes saiba quanto cada pod pode usar).
2. **Ter um servidor de métricas instalado** no cluster (o Metrics Server).

#### Instalando o Metrics Server no Kind

No Kind, o Metrics Server não vem instalado por padrão. Para instalá-lo:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

O Metrics Server no Kind pode precisar de uma configuração adicional para funcionar sem TLS. Edite o deployment do metrics-server:

```bash
kubectl -n kube-system patch deployment metrics-server --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

Aguarde o pod do Metrics Server ficar pronto:

```bash
kubectl -n kube-system get pods -l k8s-app=metrics-server
```

#### Definindo limites de recursos

Atualize o Deployment da API para incluir os limites de CPU e memória:

```yaml title="k8s/deployment.yaml (trecho)" linenums="1"
containers:
  - name: soma-api
    image: seu-usuario/soma-api
    ports:
      - containerPort: 8000
    envFrom:
      - configMapRef:
          name: soma-api-config
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

- `requests`: o mínimo garantido. O escalonador usa esse valor para decidir em qual nó alocar o pod.
- `limits`: o máximo permitido. Se o pod exceder o limite de memória, ele é encerrado (OOMKilled). Se exceder o de CPU, é limitado (throttled).

A unidade `100m` significa 100 milicores de CPU (0.1 CPU). `128Mi` significa 128 mebibytes de memória.

Aplique:

```bash
kubectl apply -f k8s/deployment.yaml
```

#### Criando o HPA

```yaml title="k8s/hpa.yaml" linenums="1"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: soma-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: soma-api
  minReplicas: 2
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Esse HPA mantém entre 2 e 8 réplicas do Deployment `soma-api`. Se o uso médio de CPU ultrapassar 50% do `requests`, o HPA adiciona réplicas. Se cair, remove.

```bash
kubectl apply -f k8s/hpa.yaml
```

Acompanhe o estado do HPA:

```bash
kubectl get hpa
```

A coluna `TARGETS` mostra o uso atual de CPU vs. o alvo (50%). Se mostrar `<unknown>/50%`, aguarde alguns minutos para o Metrics Server coletar dados.

#### Gerando carga para testar

Para observar o HPA em ação, podemos gerar carga na API. Em um terminal, mantenha o port-forward ativo:

```bash
kubectl port-forward svc/soma-api-service 8000:80
```

Em outro terminal, execute repetidamente requisições:

```bash
while true; do curl -s http://localhost:8000/sum/100/200 > /dev/null; done
```

Acompanhe o escalonamento com:

```bash
kubectl get hpa --watch
kubectl get pods --watch
```

Você verá o número de réplicas aumentar à medida que a CPU se aproxima ou ultrapassa 50%. Ao parar as requisições (Ctrl+C no loop), o HPA reduz gradualmente o número de réplicas após alguns minutos de estabilidade.

## Organização dos arquivos

Após este capítulo, o diretório `k8s/` contém:

```
k8s/
├── pod.yaml
├── deployment.yaml
├── service.yaml
├── service-nodeport.yaml
├── postgres-deployment.yaml
├── postgres-service.yaml
├── postgres-secret.yaml
├── configmap.yaml
└── hpa.yaml
```

## Exercícios

1. Crie um ConfigMap com ao menos 3 variáveis de configuração. Injete todas no Deployment da API usando `envFrom`. Acesse um pod e confirme com `env` que as variáveis estão presentes.

2. Crie um Secret com as credenciais do PostgreSQL. Atualize o Deployment do banco para usar `secretRef` em vez de variáveis em texto. Verifique que o banco continua funcionando normalmente.

3. Defina limites de recursos (`requests` e `limits`) no Deployment da API. Observe o que acontece se você definir um `requests` de memória muito alto (por exemplo, `2Gi` em um cluster Kind com pouca RAM) — o pod é alocado?

4. (Proposta aberta) Instale o Metrics Server no cluster Kind, crie o HPA e gere carga na API. Observe o escalonamento automático e documente o comportamento: quantas réplicas foram criadas, quanto tempo levou para escalar, e quanto tempo para reduzir após parar a carga.
