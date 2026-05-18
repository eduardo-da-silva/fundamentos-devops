# Repositório GitOps com Kustomize

Agora vamos criar um **novo repositório** para armazenar apenas os manifests Kubernetes da aplicação backend.

- Repositório da aplicação: <http://github.com/eduardo-da-silva/registro-atividades-backend-devops>
- Repositório GitOps: você criará nesta etapa (exemplo: `registro-atividades-k8s`)

## Por que separar repositórios?

Separar app e GitOps melhora organização e segurança:

- App repo: código, testes e Dockerfile
- GitOps repo: estado desejado do cluster

Assim, o ArgoCD observa somente o repo GitOps.

## O que é Kustomize?

Kustomize é uma ferramenta nativa do ecossistema Kubernetes para personalizar manifests sem template engine externa.

No arquivo `kustomization.yaml`, definimos:

- Lista de recursos (`resources`)
- Transformações (como troca de imagem/tag em `images`)

Isso permite que o pipeline altere apenas a tag da imagem, mantendo o restante dos manifests estável e versionado.

## Estrutura sugerida do repositório GitOps

```text
registro-atividades-k8s/
  k8s/
    namespace.yaml
    postgres-pvc.yaml
    postgres-deployment.yaml
    postgres-service.yaml
    api-deployment.yaml
    api-service.yaml
    secret.yaml.example
    .gitignore
    kustomization.yaml
```

## 1. Namespace

```yaml title="k8s/namespace.yaml"
apiVersion: v1
kind: Namespace
metadata:
  name: registro-atividades
```

## 2. Secret (modelo sem credenciais reais)

```yaml title="k8s/secret.yaml.example"
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: registro-atividades
type: Opaque
stringData:
  POSTGRES_DB: registro_atividades
  POSTGRES_USER: app_user
  POSTGRES_PASSWORD: troque_esta_senha
  DATABASE_URL: postgresql+psycopg://app_user:troque_esta_senha@postgres-service:5432/registro_atividades
```

```gitignore title="k8s/.gitignore"
secret.yaml
```

!!! danger "Nunca comite credenciais reais"
    Crie um arquivo local `k8s/secret.yaml` com valores reais e aplique no cluster manualmente.

Aplicar secret real no cluster:

```bash
kubectl apply -f k8s/secret.yaml
```

## 3. PostgreSQL

```yaml title="k8s/postgres-pvc.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: registro-atividades
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

```yaml title="k8s/postgres-deployment.yaml"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: registro-atividades
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
          image: postgres:17
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: POSTGRES_DB
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: POSTGRES_PASSWORD
          volumeMounts:
            - name: postgres-data
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-data
          persistentVolumeClaim:
            claimName: postgres-pvc
```

```yaml title="k8s/postgres-service.yaml"
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: registro-atividades
spec:
  selector:
    app: postgres
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
  type: ClusterIP
```

## 4. API backend

```yaml title="k8s/api-deployment.yaml"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registro-atividades-api
  namespace: registro-atividades
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registro-atividades-api
  template:
    metadata:
      labels:
        app: registro-atividades-api
    spec:
      containers:
        - name: api
          image: eduardosilvasc/registro-atividades-backend:latest
          ports:
            - containerPort: 8001
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: DATABASE_URL
```

```yaml title="k8s/api-service.yaml"
apiVersion: v1
kind: Service
metadata:
  name: registro-atividades-api-service
  namespace: registro-atividades
spec:
  selector:
    app: registro-atividades-api
  ports:
    - protocol: TCP
      port: 8001
      targetPort: 8001
  type: ClusterIP
```

## 5. Kustomization

```yaml title="k8s/kustomization.yaml"
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: registro-atividades
resources:
  - namespace.yaml
  - postgres-pvc.yaml
  - postgres-deployment.yaml
  - postgres-service.yaml
  - api-deployment.yaml
  - api-service.yaml
images:
  - name: eduardosilvasc/registro-atividades-backend
    newName: eduardosilvasc/registro-atividades-backend
    newTag: latest
```

## 6. Testar manifests antes do ArgoCD

Valide localmente:

```bash
kubectl apply -k k8s
kubectl get pods -n registro-atividades
kubectl get svc -n registro-atividades
```

Ajuste eventuais erros agora. Quando o ArgoCD entrar no fluxo, ele aplicará exatamente esses arquivos.
