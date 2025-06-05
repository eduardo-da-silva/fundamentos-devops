## Laboratório de integração do Packer + Vagrant + VirtualBox

Vamos fazer um laboratório de integração do Packer com o Vagrant e o VirtualBox. O objetivo é criar uma imagem de máquina virtual personalizada usando o Packer, que será utilizada pelo Vagrant para provisionar ambientes de desenvolvimento consistentes.

### Fundamentos das ferramentas

- **Packer**: É uma ferramenta de código aberto para criar imagens de máquina virtual idempotentes e reutilizáveis. Ele permite que você defina a configuração da imagem em um arquivo JSON ou HCL, especificando os recursos e as etapas de provisionamento necessários. O Packer suporta vários provedores de virtualização, incluindo VirtualBox, AWS, Azure e outros. Foi criado pela HashiCorp, a mesma empresa por trás do Terraform.
- **Vagrant**: É uma ferramenta de código aberto para criar e gerenciar ambientes de desenvolvimento virtualizados. Ele permite que você defina a configuração do ambiente em um arquivo Vagrantfile, especificando o sistema operacional, as dependências e as configurações necessárias. O Vagrant é amplamente utilizado para criar ambientes de desenvolvimento consistentes e reproduzíveis, facilitando o trabalho em equipe e a colaboração entre desenvolvedores. Foi criado também pela HashiCorp.
- **VirtualBox**: É um software de virtualização de código aberto que permite criar e executar máquinas virtuais em diferentes sistemas operacionais. Ele é amplamente utilizado como provedor de virtualização para o Vagrant, permitindo que os desenvolvedores criem e gerenciem ambientes de desenvolvimento virtualizados de forma fácil e eficiente. O VirtualBox suporta uma ampla variedade de sistemas operacionais convidados e é compatível com várias plataformas, incluindo Windows, macOS e Linux. Ele foi desenvolvido pela Oracle e é uma das ferramentas de virtualização mais populares no mundo do desenvolvimento de software.

### Pré-requisitos

1. **Packer**: Certifique-se de ter o Packer instalado em sua máquina. Você pode baixar a versão mais recente do Packer em [packer.io](https://www.packer.io/downloads).
2. **Vagrant**: Instale o Vagrant em sua máquina. Você pode encontrar as instruções de instalação em [vagrantup.com](https://www.vagrantup.com/downloads).
3. **VirtualBox**: Instale o VirtualBox, que é o provedor de virtualização utilizado pelo Vagrant. Você pode baixar o VirtualBox em [virtualbox.org](https://www.virtualbox.org/wiki/Downloads).

### Criando uma imagem personalizada com o Packer

1.  **Crie um diretório para o projeto**:

    O arquivo deve ser:

    ```bash
    mkdir packer-vagrant-virtualbox
    cd packer-vagrant-virtualbox
    ```

2.  **Crie um arquivo de configuração do Packer**

    Crie um arquivo chamado `packer.pkr.hcl` com o seguinte conteúdo:

    ```hcl
    packer {
        required_plugins {
            virtualbox = {
                version = "~> 1"
                source  = "github.com/hashicorp/virtualbox"
            }
            vagrant = {
                version = ">= 1.1.1"
                source = "github.com/hashicorp/vagrant"
            }
        }
    }
    ```

    Nesse caso estamos utilizando o HCL (HashiCorp Configuration Language) para definir a configuração do Packer. O arquivo especifica os plugins necessários, como o VirtualBox e o Vagrant. Os plugins são responsáveis por fornecer suporte a diferentes provedores de virtualização e ferramentas de provisionamento. Mais informações sobre o HCL podem ser encontradas na [documentação do Packer](https://developer.hashicorp.com/packer/integrations).

    Você precisar agora inicializar o projeto Packer e instalar o plugins registrados. Para isso, execute os seguintes comandos:

    ```bash
    packer init .
    packer install plugin github.com/hashicorp/virtualbox
    packer install plugin github.com/hashicorp/vagrant
    ```

3.  **Crie um arquivo de configuração do Packer**

    Crie um arquivo chamado `debian.json` com o seguinte conteúdo:

    ```json
    {
      "variables": {
        "vm_name": "debian12",
        "iso_url": "https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.11.0-amd64-netinst.iso",
        "iso_checksum": "sha256:30ca12a15cae6a1033e03ad59eb7f66a6d5a258dcf27acd115c2bd42d22640e8"
      },
      "builders": [
        {
          "type": "virtualbox-iso",
          "guest_os_type": "Debian_64",
          "vm_name": "{{user `vm_name`}}",
          "iso_url": "{{user `iso_url`}}",
          "iso_checksum": "{{user `iso_checksum`}}",
          "ssh_username": "vagrant",
          "ssh_password": "vagrant",
          "ssh_timeout": "20m",
          "shutdown_command": "echo 'vagrant' | sudo -S shutdown -P now",
          "disk_size": 61440,
          "http_directory": "http",
          "boot_wait": "5s",
          "boot_command": [
            "<esc><wait>",
            "auto url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg ",
            "debian-installer=en_US auto locale=pt_BR ",
            "kbd-chooser/method=us ",
            "hostname={{user `vm_name`}} ",
            "fb=false debconf/frontend=noninteractive ",
            "keyboard-configuration/layout=Brazilian ",
            "keyboard-configuration/variant=Brazilian<enter>"
          ],
          "vboxmanage": [
            ["modifyvm", "{{.Name}}", "--memory", "2048"],
            ["modifyvm", "{{.Name}}", "--cpus", "2"],

            ["modifyvm", "{{.Name}}", "--nic1", "nat"],
            ["modifyvm", "{{.Name}}", "--nictype1", "82540EM"],
            ["modifyvm", "{{.Name}}", "--cableconnected1", "on"]
          ]
        }
      ],
      "provisioners": [
        {
          "type": "shell",
          "inline": [
            "echo 'Provisionando como vagrant...'",
            "echo 'auto enp0s8' | sudo tee -a /etc/network/interfaces",
            "echo 'iface enp0s8 inet dhcp' | sudo tee -a /etc/network/interfaces",
            "mkdir -p /home/vagrant/.ssh",
            "chmod 700 /home/vagrant/.ssh",
            "curl -fsSL https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub -o /home/vagrant/.ssh/authorized_keys",
            "cat /home/vagrant/.ssh/authorized_keys",
            "chmod 600 /home/vagrant/.ssh/authorized_keys",
            "chown -R vagrant:vagrant /home/vagrant/.ssh"
          ]
        }
      ],
      "post-processors": [
        {
          "type": "vagrant",
          "output": "debian12.box"
        }
      ]
    }
    ```

    Nesse arquivo, definimos as variáveis necessárias, como o nome da máquina virtual, a URL do ISO do Debian e o checksum do ISO. Em seguida, configuramos o construtor `virtualbox-iso`, que especifica as configurações da máquina virtual, como o tipo de sistema operacional convidado, o nome da máquina virtual, a URL do ISO e as configurações de rede. O comando de inicialização é configurado para automatizar a instalação do Debian usando um arquivo de pré-configuração.

    Na chave `builder` especificamos o tipo de construtor como `virtualbox-iso`, que é responsável por criar uma imagem de máquina virtual usando o VirtualBox. As configurações incluem:

    - `guest_os_type`: Define o tipo de sistema operacional convidado, neste caso, Debian 64 bits.
    - `vm_name`: Define o nome da máquina virtual. Essa informação é passada como uma variável, permitindo que você altere facilmente o ISO sem modificar o arquivo principal.
    - `iso_url`: Especifica a URL do ISO do Debian que será usado para criar a imagem. Essa informação também é uma variável.
    - `iso_checksum`: Fornece o checksum do ISO para garantir a integridade do arquivo. Como as duas anteriores, é uma variável.
    - `ssh_username` e `ssh_password`: Definem as credenciais SSH para acessar a máquina virtual durante o provisionamento.
    - `ssh_timeout`: Define o tempo limite para a conexão SSH.
    - `shutdown_command`: Especifica o comando para desligar a máquina virtual após o provisionamento.
    - `disk_size`: Define o tamanho do disco da máquina virtual em megabytes.
    - `http_directory`: Especifica o diretório onde o Packer procurará arquivos HTTP para provisionamento.
    - `boot_wait`: Define o tempo de espera antes de iniciar o processo de boot.
    - `boot_command`: Define os comandos de boot que serão enviados para a máquina virtual durante o processo de inicialização. Esses comandos são usados para automatizar a instalação do Debian usando um arquivo de pré-configuração. Essas informações são passadas como uma lista de strings, onde cada string representa um comando a ser enviado para a máquina virtual no momento do boot.
    - `vboxmanage`: Permite modificar as configurações da máquina virtual usando comandos do VirtualBox. Aqui, estamos configurando a memória, o número de CPUs e as configurações de rede. Nesse caso, estamos configurando a máquina virtual para usar 2 GB de memória, 2 CPUs e uma interface de rede NAT.

    O bloco `provisioners` define as etapas de provisionamento que serão executadas após a criação da imagem. Neste caso, estamos usando um provisionador do tipo `shell`, que executa comandos shell na máquina virtual. Os comandos incluem a configuração da interface de rede e a adição da chave SSH do Vagrant. Note que ele baixa a chave pública do Vagrant diretamente do repositório oficial, que será usada para autenticação SSH no provisionamento.

    O bloco `post-processors` define as etapas que serão executadas após o provisionamento. Neste caso, estamos usando um post-processor do tipo `vagrant`, que cria um arquivo `.box` que pode ser usado pelo Vagrant para criar máquinas virtuais a partir da imagem criada pelo Packer.

4.  **Crie um arquivo de pré-configuração**:

    Crie um diretório chamado `http` e dentro dele, crie um arquivo chamado `preseed.cfg` com o seguinte conteúdo:

    ```cfg
    d-i debian-installer/locale string pt_BR.UTF-8

    d-i console-setup/ask_detect boolean false
    d-i keyboard-configuration/xkb-keymap select br
    d-i keyboard-configuration/layout select Brazil
    d-i keyboard-configuration/variant select abnt2
    d-i keyboard-configuration/modelcode string abnt2

    d-i netcfg/choose_interface select enp0s3
    d-i netcfg/choose_interface_if_needed boolean false
    d-i netcfg/choose_interface priority critical
    d-i netcfg/get_hostname string debian12
    d-i netcfg/get_domain string localdomain

    d-i mirror/country string manual
    d-i mirror/http/hostname string deb.debian.org
    d-i mirror/http/directory string /debian
    d-i mirror/http/proxy string

    d-i accessibility-profile/choose-profile select none

    d-i clock-setup/utc boolean true
    d-i time/zone string America/Sao_Paulo
    d-i clock-setup/ntp boolean true

    d-i partman-auto/method string regular
    d-i partman-auto/choose_recipe select atomic
    d-i partman-partitioning/confirm_write_new_label boolean true
    d-i partman/choose_partition select finish
    d-i partman/confirm boolean true
    d-i partman/confirm_nooverwrite boolean true

    d-i passwd/user-fullname string Vagrant User
    d-i passwd/username string vagrant
    d-i passwd/user-password password vagrant
    d-i passwd/user-password-again password vagrant
    d-i user-setup/encrypt-home boolean false
    d-i user-setup/allow-password-weak boolean true
    d-i passwd/user-default-groups string sudo

    d-i passwd/root-login boolean false
    d-i passwd/root-password password root
    d-i passwd/root-password-again password root

    d-i grub-installer/only_debian boolean true
    d-i grub-installer/bootdev string /dev/sda
    d-i finish-install/reboot_in_progress note
    d-i preseed/late_command string in-target adduser vagrant sudo
    d-i preseed/late_command string in-target sh -c "echo 'vagrant ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/vagrant && chmod 440 /etc/sudoers.d/vagrant"

    d-i pkgsel/run_tasksel boolean false
    d-i pkgsel/include string openssh-server build-essential
    d-i pkgsel/include string sudo curl vim net-tools ssh

    d-i pkgsel/upgrade select full-upgrade
    ```

5.  **Crie o arquivo de configuração do Vagrant**:

    Crie um arquivo chamaddo `Vagrantfile`, com seguinte conteúdo:

    ```cfg
    Vagrant.configure("2") do |config|
        config.vm.box = "debian12"

        config.vm.network "public_network", bridge: "enp2s0"

        config.vm.provider "virtualbox" do |vb|
            vb.memory = "2048"
            vb.cpus = 2
        end
    end
    ```

6.  **Crie o arquivo de imagem**

    Execute o seguinte comando para criar a sua imagem:

    ```bash
    packer build debian.json
    ```

7.  **Use o seu ambiente com o vagrant**

    Para usar o ambiente criado execute:

    ```bash
    vagrant up
    ```
