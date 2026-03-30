# Observabilidade básica

Até aqui, implantamos a aplicação, configuramos Services, externalizamos configurações e habilitamos escalonamento automático. Tudo funciona — mas como sabemos quando algo dá errado? Como investigamos falhas ou degradação de desempenho?

Observabilidade é a capacidade de entender o estado interno de um sistema a partir dos dados que ele expõe. No contexto do Kubernetes, isso envolve três pilares: **logs**, **métricas** e **traces**. Neste capítulo, cobrimos os dois primeiros com as ferramentas nativas do Kubernetes, e introduzimos os conceitos de probes — verificações de saúde que o Kubernetes usa para decidir se um contêiner está funcionando corretamente.

## Logs

O comando mais direto para inspecionar o que está acontecendo dentro de um pod é o `kubectl logs`:

```bash
kubectl logs <nome-do-pod>
```

Se o Deployment tem múltiplas réplicas e você quer ver os logs de um pod específico, liste os pods e escolha:

```bash
kubectl get pods -l app=soma-api
kubectl logs soma-api-XXXXX-YYYYY
```

Para acompanhar logs em tempo real (equivalente ao `tail -f`):

```bash
kubectl logs -f soma-api-XXXXX-YYYYY
```

Para ver logs de todos os pods de um Deployment simultaneamente:

```bash
kubectl logs -l app=soma-api --all-containers=true
```

Se o pod reiniciou por uma falha, os logs do contêiner anterior podem ser consultados com:

```bash
kubectl logs <nome-do-pod> --previous
```

## Describe e eventos

Enquanto `logs` mostra a saída da aplicação, `describe` mostra o estado do recurso na perspectiva do Kubernetes:

```bash
kubectl describe pod soma-api-XXXXX-YYYYY
```

A seção **Events** ao final é especialmente útil. Ela mostra ações como:

- `Scheduled`: em qual nó o pod foi alocado.
- `Pulling` / `Pulled`: download da imagem.
- `Started`: contêiner iniciado.
- `Unhealthy`: falha em uma verificação de saúde (probe).
- `BackOff`: o contêiner reiniciou repetidamente (crash loop).

Quando um pod não inicia, o `describe` costuma mostrar a causa antes de qualquer log existir.

## Probes: verificações de saúde

No capítulo de Docker, vimos o `HEALTHCHECK` do Dockerfile. O Kubernetes oferece um mecanismo mais granular com três tipos de probes:

| Probe              | Pergunta que responde                         | Consequência da falha                                        |
| ------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| **livenessProbe**  | O contêiner está vivo?                        | Kubernetes reinicia o contêiner.                             |
| **readinessProbe** | O contêiner está pronto para receber tráfego? | Kubernetes remove o pod do Service (não recebe requisições). |
| **startupProbe**   | O contêiner já terminou de iniciar?           | Enquanto falhar, liveness e readiness ficam desabilitados.   |

### Liveness probe

Detecta se a aplicação travou ou entrou em um estado irrecuperável. Se falhar, o Kubernetes reinicia o contêiner.

### Readiness probe

Detecta se a aplicação está pronta para receber requisições. Útil durante a inicialização (quando a aplicação está carregando dados ou conectando ao banco) ou durante picos de carga. Um pod que não está _ready_ é temporariamente removido do Service — não recebe tráfego, mas não é reiniciado.

### Exemplo prático

Atualize o Deployment da API para incluir ambas as probes:

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
    livenessProbe:
      httpGet:
        path: /
        port: 8000
      initialDelaySeconds: 10
      periodSeconds: 15
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /
        port: 8000
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 3
```

Detalhamento:

- `httpGet`: faz uma requisição HTTP GET no caminho e porta especificados. Se a resposta tiver código 200–399, a probe é considerada bem-sucedida.
- `initialDelaySeconds`: tempo de espera antes da primeira verificação (para a aplicação iniciar).
- `periodSeconds`: intervalo entre verificações.
- `failureThreshold`: número de falhas consecutivas antes de tomar a ação (reiniciar ou remover do Service).

Aplique e observe:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl describe pod $(kubectl get pods -l app=soma-api -o jsonpath='{.items[0].metadata.name}')
```

Na seção de eventos, você verá entradas como `Liveness probe succeeded` e `Readiness probe succeeded`.

### Tipos de probe

Além do `httpGet`, o Kubernetes suporta:

- `tcpSocket`: tenta abrir uma conexão TCP na porta especificada. Útil para serviços que não expõem HTTP (como bancos de dados).
- `exec`: executa um comando dentro do contêiner. A probe tem sucesso se o comando retornar código 0.

Exemplo de liveness com `tcpSocket` (para o PostgreSQL):

```yaml
livenessProbe:
  tcpSocket:
    port: 5432
  initialDelaySeconds: 15
  periodSeconds: 20
```

## Métricas com kubectl top

Se o Metrics Server estiver instalado (conforme configurado no capítulo anterior), você pode visualizar o consumo de recursos em tempo real:

```bash
kubectl top pods
kubectl top nodes
```

A saída mostra CPU e memória consumidos por cada pod ou nó. Esses são os mesmos dados que o HPA utiliza para tomar decisões de escalonamento.

## Ferramentas avançadas: próximos passos

Os comandos nativos do `kubectl` cobrem as necessidades básicas de observabilidade, mas em ambientes de produção são insuficientes. Ferramentas como **Prometheus** (coleta de métricas), **Grafana** (visualização de dashboards) e **Loki** (agregação de logs) formam uma stack de observabilidade amplamente adotada no ecossistema Kubernetes.

Essas ferramentas estão fora do escopo desta disciplina, mas o conhecimento adquirido aqui — logs, describe, probes, métricas de recursos — forma a base necessária para trabalhar com elas.

## Exercícios

1. Acesse os logs de um pod da API e faça algumas requisições via port-forward. Os logs mostram as requisições recebidas? Experimente com `kubectl logs -f` para acompanhar em tempo real.

2. Adicione liveness e readiness probes ao Deployment da API. Aplique e observe os eventos com `kubectl describe pod`. As probes estão passando?

3. Simule uma falha: altere o `path` da liveness probe para um endpoint que não existe (por exemplo, `/healthz`). Aplique e observe o comportamento — o Kubernetes reinicia o contêiner? Quantas vezes?

4. (Proposta aberta) Adicione uma liveness probe do tipo `tcpSocket` ao Deployment do PostgreSQL. Verifique com `kubectl describe` que a probe está funcionando. Depois, pare o PostgreSQL dentro do contêiner (`kubectl exec -it <pod> -- pg_ctl stop -D /var/lib/postgresql/data`) e observe se o Kubernetes reinicia o contêiner automaticamente.
