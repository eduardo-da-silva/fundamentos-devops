# Instalação automatizada do k3s e ArgoCD com Ansible

## Contexto

O tutorial de fixação preparou três servidores EC2 com o playbook `prepare-servers.yml`: pacotes instalados, usuário administrativo criado e SSH ajustado. Os servidores estão prontos, mas o Kubernetes ainda não foi instalado.

Até agora, a instalação do k3s era feita manualmente: SSH no master, rodar o script de instalação, copiar o token, SSH nos workers, rodar o script com o token. Esse processo é repetitivo e propenso a erros — um token copiado errado ou um IP trocado e o worker não entra no cluster.

Nesta aula, vamos automatizar todo o processo com um novo playbook Ansible. Ele instala o k3s no master e nos workers, coleta o token automaticamente, valida o cluster e ainda instala e expõe o ArgoCD.

## Estrutura do novo playbook

O playbook fica em `iac/ansible/playbooks/install-k3s.yml` e reutiliza o mesmo inventory do tutorial de fixação.

O inventory já inclui o campo `private_ip` em cada host — essencial para a comunicação interna entre os nós:

```ini
[control_plane]
control-plane ansible_host=18.210.10.10 private_ip=10.0.1.10

[workers]
worker-1 ansible_host=54.88.20.20 private_ip=10.0.1.11
worker-2 ansible_host=3.95.30.30 private_ip=10.0.1.12

[k8s_nodes:children]
control_plane
workers

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/minha-chave-aws.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

O playbook é dividido em três partes:

1. Instalação do k3s no control plane
2. Instalação do k3s nos workers
3. Instalação e exposição do ArgoCD

## 1. Instalação do k3s no control plane

```yaml title="playbooks/install-k3s.yml (parte 1)"
---
- name: Instalar k3s no control plane
  hosts: control_plane
  become: true
  tasks:
    - name: Instalar k3s
      shell: curl -sfL https://get.k3s.io | sh
      args:
        creates: /usr/local/bin/k3s

    - name: Aguardar arquivo de token
      wait_for:
        path: /var/lib/rancher/k3s/server/node-token
        timeout: 60

    - name: Obter token do cluster
      command: cat /var/lib/rancher/k3s/server/node-token
      register: k3s_token
      changed_when: false
```

**Explicação:**

- `hosts: control_plane`: roda apenas no nó que será o master.
- `become: true`: o script de instalação do k3s precisa de root.
- `creates: /usr/local/bin/k3s`: se o binário já existe, o Ansible pula a tarefa. Isso torna o playbook idempotente — pode executar de novo sem risco.
- `wait_for`: o token demora alguns segundos para ser gerado após a instalação. Esta tarefa aguarda até 60 segundos.
- `register: k3s_token`: guarda a saída do comando `cat` em uma variável. Essa variável será usada pelos workers.

O token fica disponível em `k3s_token.stdout`. É uma string como `K10abc123...::server:xyz...`.

## 2. Instalação do k3s nos workers

```yaml title="playbooks/install-k3s.yml (parte 2)"
- name: Instalar k3s nos workers
  hosts: workers
  become: true
  vars:
    master_private_ip: "{{ hostvars[groups['control_plane'][0]]['private_ip'] }}"
    master_token: "{{ hostvars[groups['control_plane'][0]]['k3s_token']['stdout'] | trim }}"
  tasks:
    - name: Instalar k3s agent
      shell: |
        curl -sfL https://get.k3s.io | \
          K3S_URL=https://{{ master_private_ip }}:6443 \
          K3S_TOKEN={{ master_token }} sh -
      args:
        creates: /usr/local/bin/k3s
```

**Explicação das variáveis:**

- `groups['control_plane'][0]`: o nome do primeiro (e único) host no grupo `control_plane`. Como o inventory tem apenas um master, `[0]` retorna `control-plane`.
- `hostvars[groups['control_plane'][0]]['private_ip']`: acessa a variável `private_ip` do master, definida no inventory. É o IP privado que os workers usam para se conectar ao master — a comunicação fica dentro da VPC da AWS, sem custo e mais rápida.
- `hostvars[groups['control_plane'][0]]['k3s_token']['stdout']`: acessa a saída do `cat` que registramos no playbook anterior. O `| trim` remove a quebra de linha final.

Essa é a parte mais importante do playbook: a saída de um comando (`cat` no token) vira entrada para outro (`K3S_TOKEN` no script de instalação). O Ansible faz a ponte entre os dois automaticamente.

**Sobre o `K3S_URL`:**
O worker precisa saber onde está o master. Usa-se o IP privado porque ambos estão na mesma VPC. O IP público também funcionaria, mas sairia para a internet e teria custo de transferência.

**Sobre o `creates`:**
Assim como no master, se o binário `k3s` já existir no worker, a tarefa é pulada. Isso evita reinstalar o agent se o playbook for executado novamente.

## 3. Validação do cluster

```yaml title="playbooks/install-k3s.yml (parte 3)"
- name: Validar cluster
  hosts: control_plane
  become: true
  tasks:
    - name: Aguardar nos ficarem Ready
      shell: kubectl wait --for=condition=Ready nodes --all --timeout=120s
      changed_when: false
```

Este playbook opcional espera até que todos os nós estejam com status `Ready`. Se algum nó não conseguir se conectar (token errado, IP errado, firewall bloqueando), o comando falha e o Ansible exibe o erro.

## Playbook completo

Juntando as três partes:

```yaml title="iac/ansible/playbooks/install-k3s.yml"
---
- name: Instalar k3s no control plane
  hosts: control_plane
  become: true
  tasks:
    - name: Instalar k3s
      shell: curl -sfL https://get.k3s.io | sh
      args:
        creates: /usr/local/bin/k3s

    - name: Aguardar arquivo de token
      wait_for:
        path: /var/lib/rancher/k3s/server/node-token
        timeout: 60

    - name: Obter token do cluster
      command: cat /var/lib/rancher/k3s/server/node-token
      register: k3s_token
      changed_when: false

- name: Instalar k3s nos workers
  hosts: workers
  become: true
  vars:
    master_private_ip: "{{ hostvars[groups['control_plane'][0]]['private_ip'] }}"
    master_token: "{{ hostvars[groups['control_plane'][0]]['k3s_token']['stdout'] | trim }}"
  tasks:
    - name: Instalar k3s agent
      shell: |
        curl -sfL https://get.k3s.io | \
          K3S_URL=https://{{ master_private_ip }}:6443 \
          K3S_TOKEN={{ master_token }} sh -
      args:
        creates: /usr/local/bin/k3s

- name: Validar cluster
  hosts: control_plane
  become: true
  tasks:
    - name: Aguardar nos ficarem Ready
      shell: kubectl wait --for=condition=Ready nodes --all --timeout=120s
      changed_when: false
```

## Executando o playbook

Com o inventory configurado e os servidores preparados (playbook `prepare-servers.yml` executado com sucesso):

```bash
cd iac/ansible
ansible-playbook -i inventory/hosts.ini playbooks/install-k3s.yml
```

A execução leva cerca de um minuto. O Ansible mostra o progresso de cada tarefa:

```
PLAY [Instalar k3s no control plane] ************************************

TASK [Instalar k3s] *****************************************************
changed: [control-plane]

TASK [Aguardar arquivo de token] ****************************************
ok: [control-plane]

TASK [Obter token do cluster] *******************************************
ok: [control-plane]

PLAY [Instalar k3s nos workers] *****************************************

TASK [Instalar k3s agent] ***********************************************
changed: [worker-1]
changed: [worker-2]

PLAY [Validar cluster] **************************************************

TASK [Aguardar nos ficarem Ready] ***************************************
ok: [control-plane]
```

Se tudo ocorrer bem, o cluster k3s está no ar com três nós.

## Instalação do ArgoCD

Com o cluster rodando, o próximo passo é instalar o ArgoCD. Isso poderia ser feito no mesmo playbook, mas vamos manter separado para deixar claro o que está acontecendo em cada etapa.

```yaml title="playbooks/install-argocd.yml"
---
- name: Instalar e configurar ArgoCD
  hosts: control_plane
  become: true
  tasks:
    - name: Criar namespace argocd
      shell: kubectl create namespace argocd --dry-run=client -o yaml | kubectl apply -f -
      changed_when: false

    - name: Instalar ArgoCD
      shell: kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
      args:
        creates: /usr/local/bin/argocd

    - name: Aguardar pods do ArgoCD
      shell: kubectl wait --namespace argocd --for=condition=Ready pods --all --timeout=180s
      changed_when: false

    - name: Expor ArgoCD via NodePort
      shell: kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'
      changed_when: false

    - name: Obter porta do NodePort
      shell: kubectl get svc argocd-server -n argocd -o jsonpath='{.spec.ports[0].nodePort}'
      register: argocd_port
      changed_when: false

    - name: Exibir dados de acesso
      debug:
        msg:
          - "ArgoCD instalado com sucesso"
          - "URL: https://{{ ansible_host }}:{{ argocd_port.stdout }}"
          - "Usuario: admin"
          - "Senha: kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d"
```

**Explicação:**

- O namespace é criado de forma idempotente com `--dry-run=client -o yaml | kubectl apply -f -`.
- A instalação do ArgoCD usa o manifesto oficial. O `creates: /usr/local/bin/argocd` evita reaplicar se a CLI já estiver presente — mas note que o manifesto em si pode ter mudado. Para este laboratório, é suficiente.
- O `kubectl wait` aguarda até 3 minutos para todos os pods do ArgoCD ficarem Ready. O download das imagens pode levar algum tempo.
- O `kubectl patch` altera o Service do ArgoCD de ClusterIP para NodePort, conforme discutido nas dicas extras.
- A porta alocada é capturada com `jsonpath` e exibida ao final.

Execute com:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install-argocd.yml
```

A saída mostra o IP público do master e a porta NodePort do ArgoCD:

```
TASK [Exibir dados de acesso] *******************************************
ok: [control-plane] => {
    "msg": [
        "ArgoCD instalado com sucesso",
        "URL: https://18.210.10.10:30234",
        "Usuario: admin",
        "Senha: kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d"
    ]
}
```

Acesse o ArgoCD pelo navegador em `https://<IP-PUBLICO>:<PORTA>`. O usuário padrão é `admin` e a senha é obtida rodando o comando exibido.

## Fluxo completo

Do início ao fim, a sequência de comandos é:

```bash
# 1. Provisionar infraestrutura
cd iac/terraform
terraform init
terraform plan -out tfplan
terraform apply tfplan
terraform output -json

# 2. Configurar servidores
cd ../ansible
ansible-playbook -i inventory/hosts.ini playbooks/prepare-servers.yml

# 3. Instalar k3s
ansible-playbook -i inventory/hosts.ini playbooks/install-k3s.yml

# 4. Instalar e expor ArgoCD
ansible-playbook -i inventory/hosts.ini playbooks/install-argocd.yml
```

Com isso, o ambiente completo está de pé: três servidores EC2, cluster k3s com três nós e ArgoCD acessível via navegador — tudo automatizado com Terraform e Ansible, sem um único comando manual nos servidores.

## Desafios

1. Adicione uma verificação no playbook `install-k3s.yml` para garantir que o playbook `prepare-servers.yml` já foi executado (por exemplo, verificando se o usuário `devops` existe).
2. Modifique o playbook `install-argocd.yml` para criar também um Ingress para a aplicação FastAPI, conforme o modelo da aula de dicas extras.
3. Extraia a senha do ArgoCD automaticamente e exiba no mesmo `debug` do playbook.
