# Dicas extras para IaC

## Onde estamos e o que falta

O tutorial de fixação entregou três servidores EC2 configurados, com Terraform e Ansible. O cluster k3s está rodando, as aplicações foram implantadas via ArgoCD. O acesso, no entanto, ainda depende de `kubectl port-forward` — um terminal ocupado, que precisa ser reexecutado se a conexão cair, e que exige um terminal para cada serviço.

Esta página reúne alternativas para expor serviços de forma permanente, sem depender de port-forward.

## NodePort — qualquer serviço pode ser exposto assim

O Kubernetes oferece o tipo de Service `NodePort`. Ele aloca uma porta no range 30000-32767 em todos os nós do cluster. Qualquer serviço do tipo ClusterIP pode ser convertido para NodePort com um comando.

O Security Group criado no laboratório (`k3s-cluster-sg`) já libera esse range desde o início. Nenhuma configuração extra é necessária.

### Exemplo com o ArgoCD

O ArgoCD é uma ferramenta administrativa. Não faz sentido ocupar um terminal com port-forward para ele. Basta alterar o tipo do Service:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'
```

Descubra a porta alocada:

```bash
kubectl get svc argocd-server -n argocd
```

A saída mostra algo como `443:30234/TCP`. O acesso passa a ser:

```
https://<IP-PUBLICO>:30234
```

O mesmo comando funciona para qualquer serviço. Basta trocar o nome e o namespace.

## Ingress com Traefik — aplicações web na porta 80

O k3s já vem com o **Traefik** instalado como Ingress Controller padrão. Ele escuta nas portas 80 e 443 — ambas já liberadas no Security Group.

O recurso `Ingress` do Kubernetes permite rotear requisições para serviços internos. Diferente do NodePort, não é preciso usar uma porta alta na URL.

### FastAPI no path /api

A aplicação FastAPI está rodando no serviço `registro-atividades-api-service`, namespace `registro-atividades`, porta interna 8001. Para expor via Ingress:

```yaml title="api-ingress.yaml"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: registro-atividades
spec:
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: registro-atividades-api-service
                port:
                  number: 8001
```

Aplique com:

```bash
kubectl apply -f api-ingress.yaml
```

Acesso:

```
http://<IP-PUBLICO>/api/docs
```

### E se houver um frontend?

Se no futuro uma aplicação frontend for adicionada, ela pode ocupar o path `/` enquanto a API continua em `/api`. O Ingress suporta vários paths no mesmo manifesto:

```yaml
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 3000
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: registro-atividades-api-service
                port:
                  number: 8001
```

Atenção: as chamadas do frontend para a API precisam incluir o prefixo `/api`. É um detalhe que passa despercebido quando se testa localmente e o backend está na porta 8001 direto.

### Várias estratégias, vários cenários

Não existe uma única forma de expor serviços. A escolha depende do cenário:

| Cenário | Sugestão |
|---|---|
| Ferramenta administrativa (ArgoCD, Grafana) | NodePort |
| Aplicação web única | Ingress no path `/` |
| Backend + frontend | Ingress com paths separados (`/` e `/api`) |
| Vários serviços sem hostname próprio | NodePorts diferentes (30080, 30081, ...) |

O importante é conhecer as ferramentas disponíveis e escolher a que faz sentido para cada caso.

## Nota sobre o IngressRoute do Traefik (fora do escopo)

O Traefik oferece um recurso próprio chamado `IngressRoute`, da API `traefik.containo.us/v1alpha1`. Diferente do Ingress padrão, ele permite configurar entrypoints personalizados (portas fora de 80 e 443), middlewares, rate limiting e outras funcionalidades avançadas.

Não faz parte do escopo deste laboratório, mas está na documentação oficial:

<https://doc.traefik.io/traefik/>

Uma busca por "Traefik Kubernetes IngressRoute" retorna exemplos práticos de uso.

## Comparativo das abordagens

| Abordagem | Portas | Terminal ocupado | Quantos serviços |
|---|---|---|---|
| port-forward | Qualquer | Sim | Um por terminal |
| NodePort | 30000-32767 | Não | Vários (portas diferentes) |
| Ingress padrão | 80 / 443 | Não | Vários (path ou host) |
| IngressRoute (CRD) | Qualquer | Não | Vários (configurável) |

O port-forward continua útil para testes rápidos e depuração. Para acesso permanente, NodePort e Ingress são opções mais adequadas.
