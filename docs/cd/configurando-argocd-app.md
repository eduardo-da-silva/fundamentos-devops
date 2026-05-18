# Configurando a aplicação no ArgoCD

Com ArgoCD instalado e o repo GitOps pronto, agora vamos criar o aplicativo no ArgoCD.

## 1. Criar app pela interface web

Acesse o ArgoCD em `https://SEU-IP-EC2:8080` e faça login.

Clique em `New App` e preencha:

- **Application Name**: `registro-atividades-backend`
- **Project**: `default`
- **Sync Policy**: `Automatic`
- **Repository URL**: URL do seu repositório GitOps
- **Revision**: `HEAD`
- **Path**: `k8s`
- **Cluster URL**: `https://kubernetes.default.svc`
- **Namespace**: `registro-atividades`

Em `Sync Options`, habilite:

- `Auto-Prune`
- `Self-Heal`

Essas opções garantem que recursos removidos do Git também sejam removidos do cluster e que drifts sejam corrigidos automaticamente.

## 2. Alternativa via manifesto Application

Você também pode versionar o próprio app do ArgoCD:

```yaml title="argocd-app.yaml"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: registro-atividades-backend
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/eduardo-da-silva/registro-atividades-k8s.git
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: registro-atividades
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Aplicar via `kubectl`:

```bash
kubectl apply -f argocd-app.yaml
```

## 3. Primeiro sync

Após criar o app:

1. Abra os detalhes da aplicação no ArgoCD.
2. Clique em `Sync` se necessário.
3. Verifique se todos os recursos ficam `Healthy` e `Synced`.

Validação no cluster:

```bash
kubectl get all -n registro-atividades
kubectl get pvc -n registro-atividades
```

## 4. Testar o fluxo de CD de ponta a ponta

1. Faça alteração no código do backend.
2. Push na `main` do repo da app.
3. GitHub Actions publica nova imagem e atualiza repo GitOps.
4. ArgoCD detecta novo commit e sincroniza no k3s.

Esse é o ciclo completo de GitOps em produção didática.

## 5. Acessar API após deploy

Faça port-forward no serviço da API:

```bash
kubectl port-forward svc/registro-atividades-api-service -n registro-atividades --address 0.0.0.0 8001:8001
```

Agora acesse:

- `http://SEU-IP-EC2:8001/docs`

## 6. Troubleshooting essencial

### Erro de conexão com banco

- Verifique se o Secret `db-credentials` existe:

```bash
kubectl get secret db-credentials -n registro-atividades
```

- Verifique logs da API:

```bash
kubectl logs deployment/registro-atividades-api -n registro-atividades
```

### App em `OutOfSync`

- Confira se o path da aplicação aponta para `k8s`.
- Confira se o commit mais recente realmente chegou ao repo GitOps.

### Pods reiniciando

- Verifique eventos:

```bash
kubectl get events -n registro-atividades --sort-by=.metadata.creationTimestamp
```

- Confira requests/limits e storage do laboratório.
