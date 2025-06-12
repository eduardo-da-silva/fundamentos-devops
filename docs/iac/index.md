# Infraestrutura como Código (IaC)

## O que é Infraestrutura como Código (IaC)?

A Infraestrutura como Código (Infrastructure as Code – IaC) é uma prática fundamental nas abordagens modernas de DevOps e engenharia de software. Ela permite que a infraestrutura de TI — servidores, redes, máquinas virtuais, armazenamento e até serviços de nuvem — seja gerenciada e provisionada por meio de arquivos de código, de maneira automatizada, reproduzível e versionável. Essa abordagem transforma a infraestrutura em um ativo de software, permitindo que equipes tratem a infraestrutura com os mesmos princípios e práticas que aplicam ao desenvolvimento de software.

## Origens da IaC

A prática de Infraestrutura como Código (IaC) tem suas raízes na automação de TI e na virtualização. Com o advento de tecnologias de virtualização, como VMware e Hyper-V, as equipes de TI começaram a automatizar o provisionamento de máquinas virtuais. Essa automação inicial evoluiu para a necessidade de gerenciar não apenas máquinas virtuais, mas também redes, armazenamento e outros componentes de infraestrutura de forma programática.

A popularização de serviços de nuvem, como Amazon Web Services (AWS) e Microsoft Azure, também impulsionou a adoção da IaC. Esses provedores de nuvem oferecem APIs e ferramentas que permitem que a infraestrutura seja definida e gerenciada por meio de código, tornando a IaC uma prática padrão nas operações em nuvem.

Antes do surgimento da IaC, a configuração de servidores era feita manualmente (via SSH, scripts shell, edições em tempo real de arquivos de configuração), o que era demorado, sujeito a erros e difícil de auditar ou replicar. Mesmo com a virtualização, embora ferramentas como VMWare e VirtualBox possibilitaram a criação de ambientes mais rápidos, ainda havia a dependência de interação manual. Mais recentemente, com o avanço do DevOps e da computação em nuvem, tornou-se essencial criar ambientes automatizados e reproduzíveis — daí a adoção do IaC.

## Benefícios da IaC

O uso de Infraestrutura como Código (IaC) traz uma série de benefícios significativos para as equipes de desenvolvimento e operações:

1. **Automação**: A IaC permite a automação do provisionamento e gerenciamento da infraestrutura, reduzindo a necessidade de intervenções manuais e minimizando erros.

2. **Reproduzibilidade**: Com a IaC, é possível reproduzir ambientes de forma consistente, garantindo que as mesmas configurações sejam aplicadas em diferentes ambientes (desenvolvimento, teste, produção).

3. **Versionamento**: Assim como o código-fonte, a infraestrutura pode ser versionada, permitindo que equipes rastreiem alterações, revertam para versões anteriores e colaborem de forma mais eficaz.

4. **Escalabilidade**: A IaC facilita a escalabilidade da infraestrutura, permitindo que recursos sejam provisionados ou desprovisionados rapidamente, conforme a demanda.

5. **Integração Contínua e Entrega Contínua (CI/CD)**: A IaC é uma parte essencial das práticas de CI/CD, permitindo que a infraestrutura seja tratada como código e integrada aos pipelines de entrega de software.

6. **Auditoria e Conformidade**: Com a IaC, é possível auditar as configurações da infraestrutura de forma mais eficaz, garantindo que as políticas de conformidade sejam seguidas e facilitando a identificação de desvios.

7. **Redução de error humanos**: A automação e a padronização proporcionadas pela IaC reduzem significativamente a probabilidade de erros humanos, que são comuns em processos manuais de configuração.

## Limitações da IaC

Apesar dos muitos benefícios, a Infraestrutura como Código (IaC) também apresenta algumas limitações e desafios:

1. **Complexidade**: A IaC pode introduzir complexidade adicional, especialmente em ambientes grandes e distribuídos, onde a gestão de dependências e configurações pode se tornar desafiadora.

2. **Curva de Aprendizado**: Para equipes que não estão familiarizadas com a IaC, pode haver uma curva de aprendizado significativa para entender as ferramentas e práticas associadas.

3. **Dependência de Ferramentas**: A IaC geralmente depende de ferramentas específicas, o que pode levar a um acoplamento indesejado entre a infraestrutura e as ferramentas utilizadas.

4. **Gerenciamento de Estado**: Em algumas ferramentas de IaC, o gerenciamento do estado da infraestrutura pode ser complicado, especialmente em ambientes dinâmicos onde os recursos são frequentemente criados e destruídos.

5. **Segurança**: A IaC pode introduzir riscos de segurança se não for gerenciada adequadamente, especialmente se as credenciais e segredos forem armazenados em código ou repositórios de forma insegura.

## Classificação da IaC quanto à abordagem de implementação

A Infraestrutura como Código (IaC) pode ser classificada em duas categorias principais:

1. **Declarativa**: Nesse modelo, o usuário descreve o estado desejado da infraestrutura, e a ferramenta de IaC se encarrega de criar ou modificar os recursos para alcançar esse estado. Exemplos de ferramentas incluem Terraform e AWS CloudFormation.

2. **Imperativa**: Nesse modelo, o usuário especifica os passos necessários para alcançar o estado desejado da infraestrutura. A ferramenta executa esses passos em ordem. Exemplos de ferramentas incluem Ansible e Chef.

## Tipos de ferramentas de IaC

As ferramentas de Infraestrutura como Código (IaC) podem ser divididas em duas categorias principais:

1. **Ferramentas de Provisionamento**: Essas ferramentas são usadas para criar e gerenciar recursos de infraestrutura, como máquinas virtuais, redes e serviços em nuvem. Elas permitem que os usuários definam a infraestrutura desejada em arquivos de configuração e automatizem o processo de provisionamento. Exemplos incluem Terraform, AWS CloudFormation e Pulumi.

2. **Ferramentas de Configuração**: Essas ferramentas são usadas para configurar e gerenciar o software e as aplicações em execução na infraestrutura provisionada. Elas permitem que os usuários definam o estado desejado do software e automatizem a configuração dos sistemas. Exemplos incluem Ansible, Chef e Puppet.

3. **Ferramentas de Orquestração**: Essas ferramentas são usadas para coordenar e gerenciar a execução de várias tarefas e processos em um ambiente de infraestrutura. Elas ajudam a garantir que as dependências sejam atendidas e que os recursos sejam provisionados e configurados na ordem correta. Exemplos incluem Kubernetes (para orquestração de contêineres) e Apache Mesos.

4. **Ferramentas de Empacotamento**: Essas ferramentas são usadas para empacotar e distribuir a infraestrutura como código, facilitando o compartilhamento e a reutilização de configurações. Elas permitem que os usuários criem pacotes de infraestrutura que podem ser facilmente implantados em diferentes ambientes. É muito comum para a criação de ambientes locais. Exemplos incluem Docker (para contêineres) e Vagrant (para ambientes de desenvolvimento).

## Ferramentas Comuns de IaC

Existem várias ferramentas populares que suportam a prática de IaC, incluindo:

- **Terraform**: Uma ferramenta de código aberto que permite definir e provisionar infraestrutura em nuvem usando uma linguagem de configuração declarativa.

- **AWS CloudFormation**: Um serviço da Amazon Web Services que permite criar e gerenciar recursos da AWS usando arquivos de modelo.

- **Ansible**: Uma ferramenta de automação que pode ser usada para configurar e gerenciar infraestrutura de forma declarativa.

- **Pulumi**: Uma plataforma que permite definir a infraestrutura usando linguagens de programação convencionais, como JavaScript, TypeScript e Python.

--8<-- "docs/iac/packer-vagrant-virtualbox.md"
--8<-- "docs/iac/lab-ansible.md"

## Conclusão

A Infraestrutura como Código (IaC) é uma prática essencial para equipes de DevOps que buscam aumentar a eficiência, reduzir erros e melhorar a colaboração. Ao tratar a infraestrutura como código, as organizações podem aproveitar os benefícios da automação, reproduzibilidade e versionamento, tornando-se mais ágeis e responsivas às necessidades de negócios em constante mudança.

Note que é importante escolher a ferramenta de IaC certa com base nas necessidades específicas da equipe e do projeto, considerando fatores como a complexidade da infraestrutura, a familiaridade da equipe com as ferramentas e os requisitos de integração com outras práticas de DevOps. Também, é muito comum que as equipes utilizem uma combinação de ferramentas para atender a diferentes aspectos da IaC, como provisionamento, configuração e orquestração.
