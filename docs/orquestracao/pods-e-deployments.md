# Pods e Deployments

Com o cluster Kind em execução e a imagem Docker do nosso projeto publicada (ou carregada localmente), podemos começar a implantar a aplicação no Kubernetes. Neste capítulo, trabalhamos com os dois recursos mais fundamentais: Pods e Deployments.

## Pod

O Pod é a menor unidade de implantação no Kubernetes. Ele representa um ou mais contêineres que compartilham o mesmo espaço de rede (mesmo IP) e, opcionalmente, volumes de armazenamento. Na prática, a maioria dos pods contém um único contêiner — mas o Kubernetes permite agrupar contêineres que precisam funcionar juntos (por exemplo, um contêiner principal e um sidecar de coleta de logs).

### Criando um pod

Dentro do diretório do projeto, crie uma pasta `k8s` para armazenar os manifests do Kubernetes:

```bash
mkdir -p k8s
```

Crie um arquivo `pod.yaml` com o seguinte conteúdo:

```yaml title="k8s/pod.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: soma-api
  labels:
    app: soma-api
spec:
  containers:
    - name: soma-api
      image: seu-usuario/soma-api
      ports:
        - containerPort: 8000
```

Substitua `seu-usuario` pelo seu nome de usuário no DockerHub. Se estiver usando uma imagem carregada localmente no Kind, adicione `imagePullPolicy: Never` ao contêiner para evitar que o Kubernetes tente baixar do registro remoto:

```yaml
imagePullPolicy: Never
```

Aplique o manifest:

```bash
kubectl apply -f k8s/pod.yaml
```

Verifique se o pod foi criado:

```bash
kubectl get pods
```

A saída deve mostrar o pod `soma-api` com status `Running`. Se o status for `ErrImagePull` ou `ImagePullBackOff`, verifique se a imagem está correta — ou se você carregou a imagem no Kind com `kind load docker-image`.

### Acessando a aplicação via port-forward

O pod está rodando dentro do cluster, que por padrão é isolado da rede da sua máquina. Para acessar a aplicação durante o desenvolvimento, use o port-forward:

```bash
kubectl port-forward pod/soma-api 8000:8000
```

Agora acesse http://localhost:8000 e http://localhost:8000/sum/3/4 para verificar que a API responde corretamente.

### Inspecionando o pod

Alguns comandos úteis para investigar o estado de um pod:

```bash
kubectl describe pod soma-api    # eventos, condições, IP, nó alocado
kubectl logs soma-api            # saída padrão do contêiner
kubectl exec -it soma-api -- sh  # abre um shell dentro do contêiner
```

O `describe` é particularmente útil quando o pod não inicia — ele mostra eventos como falha ao baixar a imagem, falta de recursos, ou erros de configuração.

### Limitações do pod isolado

Um pod criado diretamente (como fizemos acima) tem limitações importantes:

- Se o pod falhar ou for excluído, ele **não é recriado automaticamente**.
- Não há mecanismo de escalonamento — existe apenas uma instância.
- Atualizações de versão exigem excluir o pod e criar outro manualmente.

Esses problemas são resolvidos pelo Deployment.

## Deployment

O Deployment é um recurso de nível superior que gerencia pods de forma declarativa. Você define o estado desejado — quantidade de réplicas, imagem, configuração — e o Kubernetes se encarrega de manter esse estado. Se um pod falhar, o Deployment cria outro automaticamente. Se você atualizar a imagem, o Deployment faz a substituição gradual (rolling update).

### Criando um deployment

Antes de criar o Deployment, remova o pod isolado que criamos anteriormente para evitar conflito de nomes:

```bash
kubectl delete -f k8s/pod.yaml
```

Crie o arquivo `deployment.yaml`:

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
```

Vamos entender cada parte:

- `replicas: 3`: o Kubernetes garantirá que existam sempre 3 pods rodando.
- `selector.matchLabels`: define como o Deployment identifica os pods que ele gerencia — pelo label `app: soma-api`.
- `template`: é o modelo usado para criar os pods. Cada pod criado por este Deployment terá os labels e a spec definidos aqui.

Aplique o deployment:

```bash
kubectl apply -f k8s/deployment.yaml
```

Verifique os recursos criados:

```bash
kubectl get deployments
kubectl get pods
```

Você deve ver 3 pods com nomes no formato `soma-api-XXXXX-YYYYY`, todos com status `Running`.

### Self-healing: recuperação automática

Uma das propriedades mais importantes do Deployment é a recuperação automática. Para demonstrar, vamos excluir um pod manualmente e observar o que acontece:

```bash
kubectl get pods
```

Copie o nome de um dos pods e exclua-o:

```bash
kubectl delete pod soma-api-XXXXX-YYYYY
```

Imediatamente após excluir, liste os pods novamente:

```bash
kubectl get pods
```

Você verá que o Kubernetes já criou um novo pod para manter as 3 réplicas. O pod excluído foi substituído automaticamente. Esse comportamento é o que torna o Deployment adequado para ambientes de produção — falhas individuais são tratadas sem intervenção manual.

### Alterando o número de réplicas

Você pode alterar o número de réplicas editando o arquivo e reaplicando:

```yaml
spec:
  replicas: 5
```

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get pods
```

Agora haverá 5 pods rodando. Reduzir o número funciona da mesma forma — o Kubernetes encerrará os pods excedentes.

Também é possível alterar via comando, sem editar o arquivo:

```bash
kubectl scale deployment soma-api --replicas=2
kubectl get pods
```

Essa abordagem é útil para testes rápidos, mas em produção o ideal é manter a configuração no arquivo YAML (versionado no Git).

### Atualizando a versão da aplicação

Para atualizar a imagem do Deployment — por exemplo, após publicar uma nova versão — edite o campo `image` no `deployment.yaml` e reaaplique:

```yaml
image: seu-usuario/soma-api:v2
```

```bash
kubectl apply -f k8s/deployment.yaml
```

O Kubernetes fará um rolling update: cria novos pods com a nova imagem, espera que fiquem saudáveis, e depois encerra os antigos. Em nenhum momento todas as réplicas ficam indisponíveis simultaneamente.

Acompanhe o progresso com:

```bash
kubectl rollout status deployment soma-api
```

Se algo der errado (a nova imagem não funciona, por exemplo), você pode reverter para a versão anterior:

```bash
kubectl rollout undo deployment soma-api
```

## Organização dos arquivos

Neste ponto, a estrutura do diretório do projeto ficou assim:

```
projeto/
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── .gitignore
├── kind.yaml
└── k8s/
    ├── pod.yaml
    └── deployment.yaml
```

O diretório `k8s/` concentra todos os manifests do Kubernetes. Nos próximos capítulos, adicionaremos arquivos de Service, ConfigMap e Secret nesse mesmo diretório.

## Exercícios

1. Crie um pod isolado (sem Deployment) para a sua aplicação. Acesse-o via port-forward e confirme que a API responde. Depois, exclua o pod com `kubectl delete pod`. Execute `kubectl get pods` em seguida — o pod voltou? Por quê?

2. Crie um Deployment com 3 réplicas. Exclua um dos pods manualmente e observe o comportamento. Quanto tempo o Kubernetes leva para criar o substituto?

3. Altere o número de réplicas do Deployment para 1 e depois para 5, observando a criação e remoção de pods a cada alteração.

4. (Proposta aberta) Faça uma alteração na aplicação (por exemplo, mude a mensagem da raiz de "API de soma funcionando" para "API de soma v2"). Reconstrua a imagem com uma nova tag, publique-a (ou carregue no Kind), atualize o Deployment e acompanhe o rolling update com `kubectl rollout status`.
