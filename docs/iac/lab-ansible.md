## Laboratório de Ansible

Ansible é uma ferramenta de automação de TI que permite gerenciar e configurar sistemas, implantar aplicativos e orquestrar tarefas complexas de forma simples e eficiente. Ele utiliza uma abordagem declarativa, onde o usuário define o estado desejado da infraestrutura ou dos sistemas, e o Ansible se encarrega de aplicar as mudanças necessárias para alcançar esse estado.

Ansible é amplamente utilizado em ambientes de DevOps e Infraestrutura como Código (IaC) devido à sua simplicidade, flexibilidade e capacidade de automação. Ele é especialmente útil para gerenciar grandes quantidades de servidores e serviços, permitindo que as equipes de TI automatizem tarefas repetitivas e se concentrem em atividades mais estratégicas.

### Pré-requisitos

Antes de começar a trabalhar com Ansible, é necessário ter um ambiente configurado. Aqui estão os pré-requisitos:

1. **Sistema Operacional**: Ansible é compatível com sistemas baseados em Unix/Linux. Certifique-se de ter um sistema operacional adequado instalado, como Ubuntu, CentOS ou Debian.
2. **Python**: Ansible requer Python 3.6 ou superior. Verifique se o Python está instalado no seu sistema executando o comando `python3 --version`.
3. **Acesso SSH**: Ansible utiliza SSH para se conectar aos servidores remotos. Certifique-se de ter acesso SSH configurado para os servidores que deseja gerenciar.
4. **Instalação do Ansible**: Você pode instalar o Ansible usando o gerenciador de pacotes do seu sistema.

Note que você precise dos servidores que deseja gerenciar com Ansible. Para este laboratório, você pode usar máquinas virtuais (que criamos no laboratório anterior) ou servidores na nuvem.

Além disso, você deve gerar uma chave SSH para autenticação sem senha. Você pode fazer isso com o comando `ssh-keygen` e copiar a chave pública para os servidores remotos usando `ssh-copy-id`.

### Laboratório Prático

Neste laboratório, vamos criar um playbook simples do Ansible para instalar e configurar um servidor web Nginx em uma máquina virtual. O objetivo é demonstrar como o Ansible pode ser usado para automatizar a configuração de servidores. Sugiro que você faça isso dentro de uma pasta chamada `ansible` dentro do diretório `devops`. Isso é importante para dividir as configurações do Ansible de outras ferramentas e práticas de DevOps, com o `Packer`, `Vagrant` e `Docker`.

1.  **_Criar um arquivo de inventário_**

    Crie um arquivo chamado `hosts` na pasta `ansible` e adicione o seguinte conteúdo:

    ```ini
    [webservers]
    webserver1 ansible_host=192.168.56.101
    webserver2 ansible_host=192.168.56.102

    [all:vars]
    ansible_ssh_private_key_file=/caminho/para/chave/privada/ssh
    ansible_user=vagrant
    ```

    Substitua os endereços IP pelos endereços dos seus servidores. Esses IPs podem ser os endereços das máquinas virtuais criadas no laboratório anterior.

2.  **_Criar um playbook do Ansible_**

    Crie um arquivo chamado `install_nginx.yml` na pasta `ansible` e adicione o seguinte conteúdo:

    ```yaml
    ---
    - name: Instalar e configurar Nginx
      hosts: webservers
      become: yes
      tasks:
        - name: Instalar o Nginx
          apt:
            name: nginx
            state: present
            update_cache: yes

        - name: Garantir que o Nginx esteja rodando
          service:
            name: nginx
            state: started
    ```

    Você pode fazer várias outras configurações, como adicionar um arquivo de configuração personalizado ou copiar arquivos estáticos para o servidor. Também é possível adicionar outras tarefas, como instalar pacotes adicionais ou configurar o firewall.

3.  **_Executar o playbook do Ansible_**

    Para executar o playbook, abra um terminal na pasta `ansible` e execute o seguinte comando:

    ```bash
    ansible-playbook -i hosts install_nginx.yml
    ```

    Isso irá conectar-se aos servidores definidos no arquivo de inventário e executar as tarefas definidas no playbook.
