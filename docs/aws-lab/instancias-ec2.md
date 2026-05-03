# Instâncias EC2

Antes de criar as máquinas, é importante entender dois conceitos fundamentais da infraestrutura AWS: **VPC** e **Security Groups**. Eles definem onde as nossas instâncias vão rodar e quais conexões são permitidas.

## VPC — Virtual Private Cloud

Uma **VPC** (Nuvem Privada Virtual) é uma rede virtual isolada dentro da infraestrutura da AWS. Pense nela como uma rede local (LAN) que você controla, mas que existe na nuvem. Dentro de uma VPC, você pode criar sub-redes, definir tabelas de roteamento e controlar o tráfego de entrada e saída.

Toda conta AWS já vem com uma **VPC padrão** criada em cada região. No Learner Lab, usaremos essa VPC padrão — não precisamos criar uma nova.

O ponto mais importante para nós: instâncias EC2 criadas na mesma VPC conseguem se comunicar entre si usando seus **IPs privados**, sem que o tráfego precise sair para a internet. É assim que o nó worker vai se conectar ao master: pelo IP privado, dentro da mesma VPC.

## Security Group

Um **Security Group** é um firewall virtual que controla o tráfego de entrada (_inbound_) e saída (_outbound_) de uma instância EC2. Ele funciona como uma lista de regras: você especifica quais portas e protocolos são permitidos, e de onde pode vir o tráfego.

Diferente de firewalls tradicionais, o Security Group é _stateful_: se uma conexão de saída é permitida, a resposta correspondente de entrada é automaticamente liberada, e vice-versa.

Vamos criar um Security Group único que será compartilhado pelas duas instâncias do cluster (master e worker). Isso simplifica a configuração e garante que ambas as máquinas tenham as mesmas regras de acesso.

## Criando o Security Group

No console AWS, acesse **EC2 → Security Groups → Create security group**.

Preencha:

- **Security group name**: `k3s-cluster-sg`
- **Description**: `Security group para cluster k3s`
- **VPC**: mantenha a VPC padrão selecionada

### Regras de entrada (Inbound rules)

Clique em **Add rule** e adicione as seguintes regras:

| Tipo       | Protocolo | Porta       | Origem    | Descrição                   |
| ---------- | --------- | ----------- | --------- | --------------------------- |
| SSH        | TCP       | 22          | My IP     | Acesso SSH da sua máquina   |
| HTTP       | TCP       | 80          | 0.0.0.0/0 | Tráfego HTTP externo        |
| HTTPS      | TCP       | 443         | 0.0.0.0/0 | Tráfego HTTPS externo       |
| Custom TCP | TCP       | 6443        | 0.0.0.0/0 | API do Kubernetes (kubectl) |
| Custom TCP | TCP       | 30000-32767 | 0.0.0.0/0 | NodePort services           |

!!!note "Por que liberamos cada porta?"

    - **22**: acesso SSH para gerenciar as instâncias remotamente.
    - **80/443**: para que aplicações web fiquem acessíveis pelo navegador.
    - **6443**: o servidor de API do Kubernetes escuta nessa porta. É necessária para o `kubectl` remoto.
    - **30000–32767**: range padrão dos **NodePort** do Kubernetes. Qualquer Service do tipo NodePort usará uma porta nesse intervalo.

### Regras de saída (Outbound rules)

Mantenha a regra padrão: **All traffic** para `0.0.0.0/0`. Isso permite que as instâncias façam qualquer conexão de saída — necessário para baixar imagens Docker, instalar pacotes etc.

Clique em **Create security group**.

## Criando as instâncias EC2

Vamos criar **duas instâncias**: `k3s-master` (nó de controle) e `k3s-worker` (nó de trabalho). Você pode lançá-las juntas ou uma de cada vez — o processo é o mesmo.

No console AWS, acesse **EC2 → Instances → Launch instances**.

Configure com as seguintes especificações:

- **Name**: `k3s-master` (repita o processo depois para `k3s-worker`)
- **AMI**: Ubuntu Server 26.04 LTS
- **Instance type**: `t3.medium` (2 vCPU, 4 GB RAM)

!!!warning "Por que t3.medium?"

    O k3s, mesmo sendo uma distribuição leve do Kubernetes, ainda exige recursos razoáveis para rodar o plano de controle (etcd, API server, scheduler, controller). Instâncias menores como `t2.micro` ou `t3.micro` costumam travar ou apresentar comportamento errático durante a inicialização do cluster.

- **Key pair**: selecione **vockey** (corresponde à `labsuser.pem` que você baixou)
- **Network settings**: clique em **Select existing security group** e selecione `k3s-cluster-sg`
- **Storage**: mantenha o padrão (8 GB gp3 é suficiente)

Clique em **Launch instance**. Repita para a segunda instância com o nome `k3s-worker`.

Aguarde até que ambas estejam no estado **Running** e com **2/2 checks passed** na coluna _Status check_.

## Conectando via SSH

Para cada instância, anote o **IP público** (coluna _Public IPv4 address_ na lista de instâncias).

No terminal da sua máquina local, ajuste as permissões do arquivo PEM (obrigatório — o SSH rejeita chaves com permissões abertas):

```bash
chmod 400 labsuser.pem
```

Conecte na instância master:

```bash
ssh -i labsuser.pem ubuntu@<ip-publico-master>
```

Conecte na instância worker (em outra aba do terminal):

```bash
ssh -i labsuser.pem ubuntu@<ip-publico-worker>
```

!!!warning "Os IPs públicos mudam a cada sessão"

    Sempre que você encerrar o lab (**End Lab**) e reiniciá-lo, as instâncias recebem novos IPs públicos. **Antes de cada sessão**, acesse o console EC2 e anote os IPs atuais. Os IPs **privados**, por outro lado, permanecem estáveis enquanto as instâncias existirem — e serão usados para a comunicação interna entre os nós.
