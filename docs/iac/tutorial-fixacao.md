# Tutorial de fixação (lab)

Este tutorial consolida os conceitos da trilha com um laboratório guiado, funcional e executável do início ao fim.

## Objetivo

Provisionar 3 instâncias EC2 com Terraform e configurar os servidores com Ansible, validando conectividade SSH.

## Cenário do laboratório

- AWS Learner Lab
- 1 instância control-plane
- 2 instâncias workers
- Ubuntu Server
- Security Group com SSH restrito ao seu IP
- Comunicação interna liberada entre os nós

## Pré-requisitos

- Learner Lab ativo
- Terraform instalado
- Ansible instalado
- Chave SSH criada/importada na AWS (campo `key_name`)

## Etapa 0 - Preparar credenciais do Learner Lab

No AWS Learner Lab, abra **AWS Details** e copie:

- AWS Access Key ID
- AWS Secret Access Key
- AWS Session Token
- Região (exemplo: `us-east-1`)

Exporte no terminal:

```bash
export AWS_ACCESS_KEY_ID="SEU_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="SEU_SECRET_KEY"
export AWS_SESSION_TOKEN="SEU_SESSION_TOKEN"
export AWS_DEFAULT_REGION="us-east-1"
```

Valide:

```bash
aws sts get-caller-identity
```

## Etapa 1 - Criar estrutura de pastas

No diretório do repositório, crie:

```bash
mkdir -p iac/terraform iac/ansible/{inventory,group_vars,playbooks}
```

Estrutura esperada:

```text
iac/
   terraform/
      main.tf
      variables.tf
      outputs.tf
      terraform.tfvars
      terraform.tfvars.example
   ansible/
      inventory/
         hosts.ini
      group_vars/
         all.yml
      playbooks/
         prepare-servers.yml
```

## Etapa 2 - Criar os arquivos Terraform

### Arquivo `iac/terraform/variables.tf`

```hcl
variable "aws_region" {
   description = "Região AWS"
   type        = string
   default     = "us-east-1"
}

variable "project_name" {
   description = "Prefixo para nome e tags dos recursos"
   type        = string
   default     = "fundamentos-devops"
}

variable "ami_id" {
   description = "AMI Ubuntu Server"
   type        = string
}

variable "instance_type" {
   description = "Tipo da instância EC2"
   type        = string
   default     = "t3.micro"
}

variable "key_name" {
   description = "Nome da chave SSH cadastrada na AWS"
   type        = string
}

variable "allowed_ssh_cidr" {
   description = "IPs com eacesso ao SSH, ex: 203.0.113.10/32 ou 0.0.0.0/0"
   type        = string
}
```

### Arquivo `iac/terraform/main.tf`

```hcl
terraform {
   required_version = ">= 1.6.0"

   required_providers {
      aws = {
         source  = "hashicorp/aws"
         version = "~> 5.0"
      }
   }
}

provider "aws" {
   region = var.aws_region
}

data "aws_vpc" "default" {
   default = true
}

data "aws_subnets" "default" {
   filter {
      name   = "vpc-id"
      values = [data.aws_vpc.default.id]
   }
}

locals {
   nodes = {
      control-plane = { role = "control-plane" }
      worker-1      = { role = "worker" }
      worker-2      = { role = "worker" }
   }
}

resource "aws_security_group" "nodes_sg" {
   name        = "${var.project_name}-sg"
   description = "SSH externo e tráfego interno entre os nós"
   vpc_id      = data.aws_vpc.default.id

   ingress {
      description = "SSH do seu computador"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = [var.allowed_ssh_cidr]
   }

   ingress {
      description = "Comunicação interna entre os servidores"
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      self        = true
   }

   egress {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
   }

   tags = {
      Name = "${var.project_name}-sg"
   }
}

resource "aws_instance" "nodes" {
   for_each = local.nodes

   ami                         = var.ami_id
   instance_type               = var.instance_type
   key_name                    = var.key_name
   subnet_id                   = data.aws_subnets.default.ids[0]
   vpc_security_group_ids      = [aws_security_group.nodes_sg.id]
   associate_public_ip_address = true

   tags = {
      Name = "${var.project_name}-${each.key}"
      Role = each.value.role
   }
}
```

### Arquivo `iac/terraform/outputs.tf`

```hcl
output "public_ips" {
   description = "IPs públicos das instâncias"
   value = {
      for name, instance in aws_instance.nodes :
      name => instance.public_ip
   }
}

output "private_ips" {
   description = "IPs privados das instâncias"
   value = {
      for name, instance in aws_instance.nodes :
      name => instance.private_ip
   }
}

output "instance_ids" {
   description = "IDs das instâncias"
   value = {
      for name, instance in aws_instance.nodes :
      name => instance.id
   }
}
```

### Arquivo `iac/terraform/terraform.tfvars.example`

```hcl
aws_region       = "us-east-1"
project_name     = "fundamentos-devops"
ami_id           = "ami-xxxxxxxxxxxxxxxxx"
instance_type    = "t3.micro"
key_name         = "minha-chave-aws"
allowed_ssh_cidr = "203.0.113.10/32"
```

### Arquivo `iac/terraform/terraform.tfvars`

Copie o exemplo e preencha os valores reais:

```bash
cp iac/terraform/terraform.tfvars.example iac/terraform/terraform.tfvars
```

Preenchimentos importantes:

- `ami_id`: AMI Ubuntu da região escolhida
- `key_name`: nome exato da chave SSH na AWS
- `allowed_ssh_cidr`: seu IP público com `/32` (Sugiro usar `0.0.0.0/0` apenas para testes, não em produção)

Para descobrir seu IP público:

```bash
curl -s ifconfig.me
```

## Etapa 3 - Executar Terraform

```bash
cd iac/terraform
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out tfplan
terraform apply tfplan
```

Coletar outputs:

```bash
terraform output
terraform output -json
```

## Etapa 4 - Criar os arquivos Ansible

Volte para a raiz do projeto:

```bash
cd ../..
```

### Arquivo `iac/ansible/inventory/hosts.ini`

Substitua os IPs públicos e privados pelos valores dos outputs do Terraform:

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

### Arquivo `iac/ansible/group_vars/all.yml`

```yaml
---
common_packages:
   - curl
   - vim
   - htop
   - net-tools
   - ca-certificates
   - apt-transport-https

admin_user: devops
admin_pubkey: "ssh-ed25519 AAAA...substitua-pela-chave-publica"
```

### Arquivo `iac/ansible/playbooks/prepare-servers.yml`

```yaml
---
- name: Preparar servidores para futura instalação do Kubernetes
   hosts: k8s_nodes
   become: true
   tasks:
      - name: Atualizar cache do APT
         ansible.builtin.apt:
            update_cache: true
            cache_valid_time: 3600

      - name: Instalar pacotes básicos
         ansible.builtin.apt:
            name: "{{ common_packages }}"
            state: present

      - name: Criar usuário administrativo
         ansible.builtin.user:
            name: "{{ admin_user }}"
            groups: sudo
            append: true
            shell: /bin/bash
            create_home: true

      - name: Criar diretório .ssh do usuário administrativo
         ansible.builtin.file:
            path: "/home/{{ admin_user }}/.ssh"
            state: directory
            owner: "{{ admin_user }}"
            group: "{{ admin_user }}"
            mode: "0700"

      - name: Configurar authorized_keys do usuário administrativo
         ansible.builtin.copy:
            content: "{{ admin_pubkey }}\n"
            dest: "/home/{{ admin_user }}/.ssh/authorized_keys"
            owner: "{{ admin_user }}"
            group: "{{ admin_user }}"
            mode: "0600"

      - name: Desativar autenticação SSH por senha
         ansible.builtin.lineinfile:
            path: /etc/ssh/sshd_config
            regexp: '^#?PasswordAuthentication'
            line: 'PasswordAuthentication no'
            state: present
         notify: Reiniciar SSH

      - name: Desativar login SSH de root
         ansible.builtin.lineinfile:
            path: /etc/ssh/sshd_config
            regexp: '^#?PermitRootLogin'
            line: 'PermitRootLogin no'
            state: present
         notify: Reiniciar SSH

   handlers:
      - name: Reiniciar SSH
         ansible.builtin.service:
            name: ssh
            state: restarted
```

## Etapa 5 - Executar validações com Ansible

Ajuste permissão da chave privada:

```bash
chmod 400 ~/.ssh/minha-chave-aws.pem
```

Entrar no diretório do Ansible:

```bash
cd iac/ansible
```

Validar inventário:

```bash
ansible-inventory -i inventory/hosts.ini --graph
```

Testar conectividade:

```bash
ansible -i inventory/hosts.ini k8s_nodes -m ping
```

Executar playbook:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/prepare-servers.yml
```

## Etapa 6 - Verificação final

Teste SSH manual no control plane:

```bash
ssh -i ~/.ssh/minha-chave-aws.pem ubuntu@IP_PUBLICO_DO_CONTROL_PLANE
```

Teste ad-hoc nos três nós:

```bash
ansible all -i inventory/hosts.ini -m command -a "hostname -I"
```

## Checklist de entrega

- [ ] 3 instâncias EC2 criadas
- [ ] Security Group com SSH restrito ao seu IP
- [ ] Saída de `terraform output` com IPs públicos e privados
- [ ] `ansible -m ping` funcionando nos 3 hosts
- [ ] Playbook executado com sucesso
- [ ] Acesso SSH validado

## Limpeza do ambiente (obrigatória ao final)

Para evitar consumo indevido no Learner Lab:

```bash
cd iac/terraform
terraform destroy
```

## Desafio (sem passo a passo)

Implemente as melhorias abaixo:

1. Quantidade de workers por variável.
2. Criação dinâmica de workers com `for_each` ou `count`.
3. Inventário Ansible separado por grupos e ambientes.
4. Role de hardening SSH reutilizável.
5. Evidência final com:

```bash
ansible all -i inventory/hosts.ini -m command -a "hostname -I"
```
