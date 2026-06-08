# Ansible para configuração

Ansible é uma ferramenta de automação de configuração. Ele conecta via SSH nos servidores e aplica mudanças com playbooks YAML.

O Ansible é ideal para configurar o sistema operacional, instalar pacotes, criar usuários, ajustar arquivos de configuração e preparar os servidores para rodar aplicações ou serviços. Ele é complementar ao Terraform, que é focado em provisionar a infraestrutura. Com Ansible, você pode garantir que os servidores estejam configurados de forma consistente, seguindo as melhores práticas e evitando erros manuais.

O Ansible foi criado pelo engenheiro de software Michael DeHaan em 2012. Ele é escrito em Python e é conhecido por sua simplicidade e facilidade de uso. O Ansible usa uma abordagem declarativa, onde você descreve o estado desejado dos seus servidores e o Ansible se encarrega de aplicar as mudanças necessárias para alcançar esse estado. Ele é amplamente utilizado em ambientes de TI para automação de tarefas de configuração, gerenciamento de sistemas e orquestração de aplicações. Hoje, ele é mantido pela Red Hat e tem uma grande comunidade de usuários e contribuidores.

## Conceitos fundamentais

**Inventario**
: Lista de hosts e grupos gerenciados. Pode ser estático (arquivo) ou dinâmico (script).

**Playbook**
: Fluxo declarativo com tarefas. Define o que deve ser feito nos hosts. Pode incluir variáveis, handlers e roles. 

**Tasks**
: Ações individuais de configuração. Exemplo: instalar pacote, criar usuário, copiar arquivo.

**Modules**
: Unidades de ação prontas, como apt, user, file e service. Eles abstraem a complexidade de comandos específicos do sistema operacional.

**Roles**
: Estrutura para organizar e reutilizar automações. Permite dividir o código em partes menores e mais gerenciáveis, facilitando a manutenção e a colaboração.

**Idempotencia**
: Reexecução segura sem causar efeitos colaterais. Se o estado desejado já foi alcançado, o Ansible não fará mudanças adicionais, garantindo que a configuração seja consistente e previsível.

## Estrutura recomendada

Abaixo está uma estrutura de pastas recomendada para organizar o código Ansible:

```text
iac/
  ansible/
    ansible.cfg
    inventory/
      hosts.ini
    group_vars/
      all.yml
    playbooks/
      prepare-servers.yml
    roles/
      common/
        tasks/
          main.yml
```

No tutorial que realizaremos, vamos entender melhor cada um desses componentes e como eles se encaixam para configurar os servidores provisionados com Terraform, preparando o ambiente para a instalação do Kubernetes em etapas posteriores.

## Configurações base para o nosso cenário

- Instalação de pacotes essenciais
- Criação de usuário administrativo
- Configuração de chave pública
- Hardening básico de SSH

## Comandos essenciais

Alguns comandos essenciais para trabalhar com Ansible:

- Listar hosts e grupos do inventário: `ansible-inventory -i inventory/hosts.ini --graph`
- Testar conectividade com ping: `ansible -i inventory/hosts.ini k8s_nodes -m ping`
- Executar playbook: `ansible-playbook -i inventory/hosts.ini playbooks/prepare-servers.yml`
- Executar playbook em modo check: `ansible-playbook -i inventory/hosts.ini playbooks/prepare-servers.yml --check`


## Observações práticas

- Ajuste a permissão da chave .pem com chmod 400.
- Em timeout SSH, revise Security Group, IP e usuário.
- Se IPs mudarem, atualize o inventário antes de executar.
