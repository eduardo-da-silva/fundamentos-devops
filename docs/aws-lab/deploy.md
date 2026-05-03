# Deploy das aplicações

Com o cluster k3s em execução e os dois nós prontos, vamos implantar aplicações. Faremos dois exemplos: um simples e um avançado com banco de dados.

Todos os comandos `kubectl` devem ser executados **no master** (via SSH ou kubectl remoto configurado na seção anterior).

## Namespaces

Antes de começar o deploy, vale introduzir o conceito de **Namespace**. Um namespace é uma forma de dividir logicamente os recursos do cluster: pods, services, deployments e outros objetos ficam isolados dentro de um namespace, evitando conflitos de nomes e facilitando o gerenciamento por equipe ou ambiente (ex.: `dev`, `staging`, `production`).

Por padrão, ao não especificar um namespace, os recursos vão para o namespace `default`. Neste laboratório, vamos criar namespaces separados para cada deploy:

```bash
sudo kubectl create namespace soma
sudo kubectl create namespace app-completa
```

Verifique os namespaces criados:

```bash
sudo kubectl get namespaces
```

!!!tip "Especificando o namespace nos comandos"

    Para listar recursos em um namespace específico, adicione `-n <namespace>`:
    ```bash
    sudo kubectl get pods -n soma
    sudo kubectl get services -n app-completa
    ```

## Deploy simples — soma-api

Vamos implantar a `soma-api` — a API desenvolvida nos capítulos de Integração Contínua e publicada no DockerHub.

### Estrutura de arquivos

Crie um diretório para os manifests:

```bash
mkdir -p ~/k8s-soma
```

Crie o arquivo `~/k8s-soma/deployment.yaml` com o seguinte conteúdo:

```yaml title="k8s-soma/deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: soma-api
  namespace: soma
spec:
  replicas: 2
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
```

Crie o arquivo `~/k8s-soma/service.yaml`:

```yaml title="k8s-soma/service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: soma-api-service
  namespace: soma
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

!!!note "Substitua `seu-usuario`"

    No `deployment.yaml`, troque `seu-usuario/soma-api` pelo seu nome de usuário real no DockerHub — o mesmo usado ao publicar a imagem nos capítulos de CI/CD.

### Aplicando os manifests

```bash
sudo kubectl apply -f ~/k8s-soma/
```

Acompanhe a criação dos pods:

```bash
sudo kubectl get pods -n soma -w
```

Aguarde até todos os pods estarem `Running`. Verifique o service:

```bash
sudo kubectl get services -n soma
```

### Testando

Acesse a API pelo IP público do master (ou do worker) na porta `30080`:

```
http://<ip-publico-master>:30080/sum/10/5
```

A resposta esperada é `{"result":15}`.

!!!tip "NodePort é acessível em qualquer nó"

    No k3s, o NodePort fica disponível em **todos os nós do cluster** — master e worker. Você pode usar o IP público de qualquer um dos dois. Ambos estão no Security Group `k3s-cluster-sg` com a porta `30000-32767` liberada.

## Deploy avançado — API com banco de dados

Neste segundo exemplo, vamos implantar uma aplicação completa: uma API que persiste dados em um banco PostgreSQL, usando **ConfigMap** para configurações não sensíveis, **Secret** para credenciais e **PersistentVolumeClaim** para armazenamento persistente — seguindo as práticas da unidade de Orquestração de Contêineres.

### Estrutura de arquivos

Crie o diretório:

```bash
mkdir -p ~/k8s-app
```

### Secret — credenciais do banco

Crie o arquivo `~/k8s-app/secret.yaml`:

```yaml title="k8s-app/secret.yaml" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
  namespace: app-completa
type: Opaque
stringData:
  POSTGRES_USER: devops
  POSTGRES_PASSWORD: devops123
  POSTGRES_DB: app_db
```

### ConfigMap — configurações da API

Crie o arquivo `~/k8s-app/configmap.yaml`:

```yaml title="k8s-app/configmap.yaml" linenums="1"
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
  namespace: app-completa
data:
  DATABASE_HOST: postgres-service
  DATABASE_PORT: '5432'
  DATABASE_NAME: app_db
```

### PersistentVolumeClaim — armazenamento do PostgreSQL

Crie o arquivo `~/k8s-app/postgres-pvc.yaml`:

```yaml title="k8s-app/postgres-pvc.yaml" linenums="1"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: app-completa
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Deployment do PostgreSQL

Crie o arquivo `~/k8s-app/postgres-deployment.yaml`:

```yaml title="k8s-app/postgres-deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: app-completa
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
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
```

### Service ClusterIP para o PostgreSQL

Crie o arquivo `~/k8s-app/postgres-service.yaml`:

```yaml title="k8s-app/postgres-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: app-completa
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
```

O Service do tipo `ClusterIP` expõe o banco apenas dentro do cluster. A API o acessa pelo nome DNS `postgres-service` — que o Kubernetes resolve automaticamente dentro do namespace `app-completa`.

### Deployment da API

Crie o arquivo `~/k8s-app/api-deployment.yaml`:

```yaml title="k8s-app/api-deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-api
  namespace: app-completa
spec:
  replicas: 2
  selector:
    matchLabels:
      app: minha-api
  template:
    metadata:
      labels:
        app: minha-api
    spec:
      containers:
        - name: minha-api
          image: seu-usuario/minha-api
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef:
                name: api-config
            - secretRef:
                name: postgres-credentials
```

### Service NodePort para a API

Crie o arquivo `~/k8s-app/api-service.yaml`:

```yaml title="k8s-app/api-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: app-completa
spec:
  type: NodePort
  selector:
    app: minha-api
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30090
      protocol: TCP
```

### Aplicando todos os manifests

```bash
sudo kubectl apply -f ~/k8s-app/
```

Acompanhe a inicialização:

```bash
sudo kubectl get pods -n app-completa -w
```

Aguarde até o pod do `postgres` e os pods da `minha-api` estarem `Running`. O PostgreSQL costuma levar alguns segundos a mais para concluir a inicialização interna antes de aceitar conexões.

Verifique os services e volumes:

```bash
sudo kubectl get services -n app-completa
sudo kubectl get pvc -n app-completa
```

### Testando

Acesse a API pelo IP público na porta `30090`:

```
http://<ip-publico-master>:30090
```

Para verificar se a API consegue alcançar o banco via DNS interno, abra um shell dentro de um pod da API:

```bash
sudo kubectl exec -it \
  $(sudo kubectl get pods -n app-completa -l app=minha-api -o jsonpath='{.items[0].metadata.name}') \
  -n app-completa -- sh
```

Dentro do pod, teste a conectividade com o banco:

```bash
nc -zv postgres-service 5432
```

Uma resposta `open` confirma que a API consegue se comunicar com o PostgreSQL pelo Service ClusterIP.

## Exercícios

1. **Adicionar um terceiro nó worker**

   Crie uma terceira instância EC2 com as mesmas configurações (Ubuntu 26.04, `t3.medium`, Security Group `k3s-cluster-sg`). Recupere novamente o token do master:

   ```bash
   sudo cat /var/lib/rancher/k3s/server/node-token
   ```

   Instale o k3s agent na nova instância e verifique no master:

   ```bash
   sudo kubectl get nodes
   ```

   Depois, observe com `sudo kubectl get pods -o wide -n soma` em qual nó cada pod está alocado. O Kubernetes redistribuiu automaticamente?

   Dicas:
   - O token não muda entre sessões. O IP privado do master, porém, pode mudar se a instância for reiniciada — sempre verifique antes de adicionar um novo worker.
   - Se o nó não aparecer em `Ready`, inspecione os logs no worker: `sudo journalctl -u k3s-agent -f`.

2. **Escalar e observar a distribuição**

   Escale o Deployment da `soma-api` para 4 réplicas:

   ```bash
   sudo kubectl scale deployment soma-api -n soma --replicas=4
   ```

   Execute `sudo kubectl get pods -n soma -o wide` e observe a coluna `NODE`. Como os pods foram distribuídos entre os workers? Por que nem sempre a distribuição é perfeitamente uniforme?

3. **Deploy completo de uma aplicação do curso**

   Escolha uma aplicação desenvolvida nos capítulos anteriores (de preferência aquela com banco de dados e variáveis de ambiente), adapte os manifests do exemplo avançado para a sua imagem e faça o deploy em um novo namespace. Certifique-se de:
   - Criar um namespace dedicado.
   - Usar um Secret para todas as credenciais sensíveis.
   - Usar um ConfigMap para as variáveis de configuração não sensíveis.
   - Configurar um PVC para persistência do banco.
   - Verificar o acesso externo pelo IP público do cluster.
