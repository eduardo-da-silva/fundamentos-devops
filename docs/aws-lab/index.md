# Laboratório AWS Learner Lab

Até aqui, trabalhamos com Kubernetes localmente usando o Kind, que roda nós como contêineres Docker na nossa própria máquina. Esse ambiente é excelente para aprender e experimentar — mas não reflete como o Kubernetes funciona em produção: máquinas separadas, IPs reais, comunicação entre nós pela rede, e acesso remoto ao cluster.

Neste laboratório, vamos colocar tudo isso em prática usando o **AWS Learner Lab**, um ambiente educacional gerenciado que disponibiliza uma conta real da Amazon Web Services (AWS) por tempo limitado, sem custo para o aluno.

## O que vamos construir

Ao longo deste laboratório, vamos:

1. Acessar o AWS Learner Lab e iniciar o ambiente.
2. Criar instâncias EC2 (máquinas virtuais na nuvem) e configurar a rede para o cluster.
3. Instalar o **k3s** — uma distribuição leve do Kubernetes — em um nó master e em um nó worker.
4. Configurar o acesso remoto ao cluster via `kubectl`.
5. Implantar duas aplicações: uma simples e uma completa com banco de dados, ConfigMap, Secret e Namespace.

Ao final, teremos uma aplicação acessível pela internet a partir de um IP público da AWS, rodando em um cluster Kubernetes real com múltiplos nós.

## Sobre o AWS Learner Lab

O AWS Learner Lab é um serviço educacional oferecido pela Amazon como parte de programas como o **AWS Academy**. Ele provisiona uma conta AWS temporária e isolada para cada aluno, com créditos pré-definidos, permitindo experimentar os serviços da AWS em um ambiente real sem risco de custos inesperados.

Algumas características importantes:

- **Tempo limitado por sessão**: o lab tem um contador regressivo. Ao encerrar a sessão, todas as instâncias em execução são paradas (mas não destruídas por padrão).
- **IPs públicos mudam**: cada vez que você reinicia o lab ou as instâncias, os IPs públicos são reatribuídos. Isso tem impacto direto em como configuramos o acesso remoto ao cluster.
- **Créditos controlados**: o lab possui um saldo de créditos AWS. Deixar instâncias rodando sem necessidade consome esse saldo. Sempre encerre o lab com **End Lab** ao terminar.
- **Ambiente real**: diferente de simuladores, você está usando os mesmos serviços AWS que uma empresa real usa — EC2, VPC, Security Groups, IAM etc.

Nas próximas páginas, percorreremos cada etapa do laboratório em detalhes.

