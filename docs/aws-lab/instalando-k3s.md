# Instalando o k3s

## O que é o k3s?

O **k3s** é uma distribuição de Kubernetes certificada, leve e projetada para cenários onde os recursos de hardware são limitados ou onde se deseja simplicidade de instalação e operação. Foi criado pela **Rancher Labs** (hoje parte da SUSE) e é mantido como projeto de código aberto sob a CNCF.

Enquanto uma instalação padrão do Kubernetes (_upstream_) envolve vários componentes separados (`kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, `etcd`, `kubelet`, `kube-proxy`, CNI plugin...), o k3s empacota tudo isso em um **único binário** de menos de 100 MB. Isso reduz drasticamente a complexidade de instalação e os requisitos de memória.

O k3s usa **SQLite** como banco de dados de estado por padrão (em vez do etcd), o que o torna ainda mais leve para clusters de um único nó ou de pequena escala. Para clusters maiores ou de alta disponibilidade, é possível configurá-lo com etcd ou bancos externos.

## Por que k3s e não outras ferramentas?

Nas aulas anteriores, usamos o **Kind** (Kubernetes IN Docker), que simula nós como contêineres Docker. O Kind é excelente para desenvolvimento local, mas não é projetado para rodar em máquinas reais separadas.

Outras alternativas existem para ambientes reais:

| Ferramenta   | Uso principal                                                 | Complexidade de instalação |
| ------------ | ------------------------------------------------------------- | -------------------------- |
| **kubeadm**  | Instalação manual do Kubernetes upstream em servidores reais  | Alta                       |
| **k3s**      | Kubernetes leve para ambientes com recursos limitados ou edge | Baixa                      |
| **MicroK8s** | Distribuição leve mantida pela Canonical (Ubuntu)             | Baixa                      |
| **Minikube** | Cluster local de nó único, para desenvolvimento               | Baixa (mas só local)       |
| **RKE2**     | Distribuição da Rancher focada em segurança e conformidade    | Média                      |

Usamos o **k3s** neste laboratório porque:

- Instalação em **um único comando** — sem configuração prévia de PKI, kubeadm init, join tokens manuais etc.
- Funciona bem em instâncias `t3.medium` (o kubeadm upstream seria pesado demais para um lab educacional).
- É amplamente usado em produção em ambientes de edge computing, IoT e clusters menores.
- Já vem com o `kubectl` embutido — não precisa instalar separadamente.

## Instalando o k3s no nó master

Com o SSH aberto na instância `k3s-master`, execute o script oficial de instalação:

```bash
curl -sfL https://get.k3s.io | sudo sh
```

!!!warning "Use | sudo sh"

    É obrigatório usar `| sudo sh`. O script de instalação precisa de privilégios de root para instalar serviços no systemd, copiar o binário para `/usr/local/bin` e criar os arquivos de configuração em `/etc/rancher`. Sem o `sudo`, a instalação falhará silenciosamente ou com erros de permissão.

O script baixa o binário do k3s, instala o serviço `k3s` no systemd e inicia o servidor. Aguarde cerca de 30 segundos e verifique se o nó está pronto:

```bash
sudo kubectl get nodes
```

A saída esperada:

```
NAME        STATUS   ROLES                  AGE   VERSION
k3s-master  Ready    control-plane,master   30s   v1.x.x+k3s1
```

## Recuperando o token do cluster

Para que os workers possam se juntar ao cluster, o k3s usa um token gerado automaticamente durante a instalação. Recupere-o no master:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

A saída será algo como:

```
K10abc123def456...::server:xyz789...
```

Copie esse valor completo — você vai precisar dele no próximo passo.

## Instalando o k3s no nó worker

Abra uma **segunda sessão de terminal** e conecte na instância `k3s-worker` via SSH:

```bash
ssh -i labsuser.pem ubuntu@<ip-publico-worker>
```

Execute o script de instalação em modo **agent**, passando o endereço do master e o token:

```bash
curl -sfL https://get.k3s.io | sudo K3S_URL=https://<ip-privado-master>:6443 K3S_TOKEN=<token> sh -
```

!!!warning "Posição correta do sudo"

    O `sudo` deve ficar **imediatamente após o pipe** (`|`), antes das variáveis de ambiente `K3S_URL` e `K3S_TOKEN`. Isso garante que as variáveis sejam definidas na sessão com privilégios de root, onde o script será executado. Se você colocar as variáveis antes do `sudo` (como `K3S_URL=... K3S_TOKEN=... sudo sh`), elas **não serão repassadas** para a sessão elevada e a instalação falhará.

    Forma correta:
    ```bash
    curl -sfL https://get.k3s.io | sudo K3S_URL=https://<ip>:6443 K3S_TOKEN=<token> sh -
    ```

    Forma incorreta:
    ```bash
    # NÃO FAÇA ASSIM — as variáveis não chegam ao sudo
    curl -sfL https://get.k3s.io | K3S_URL=https://<ip>:6443 K3S_TOKEN=<token> sudo sh -
    ```

!!!tip "Use o IP privado do master"

    Para a comunicação entre os nós dentro da mesma VPC da AWS, use o **IP privado** do master (visível na coluna *Private IPv4 address* no console EC2). O IP privado não muda entre reinicializações das instâncias e o tráfego permanece dentro da rede da AWS, sem custo adicional de transferência.

## Verificando o cluster

Após a instalação do worker, volte para o terminal do master e verifique os nós:

```bash
sudo kubectl get nodes
```

Você deve ver os dois nós com status `Ready`:

```
NAME        STATUS   ROLES                  AGE   VERSION
k3s-master  Ready    control-plane,master   5m    v1.x.x+k3s1
k3s-worker  Ready    <none>                 30s   v1.x.x+k3s1
```

Se o worker demorar mais de 1-2 minutos para aparecer ou estiver em `NotReady`, verifique os logs do serviço no nó worker:

```bash
sudo journalctl -u k3s-agent -f
```

Os erros mais comuns são: IP privado do master incorreto, token copiado com espaços ou quebras de linha, ou porta 6443 não liberada no Security Group.
