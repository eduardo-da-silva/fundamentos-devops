# Docker

O Docker é uma plataforma de software que permite criar, testar e implantar aplicativos rapidamente. O Docker empacota o software em unidades padronizadas chamadas contêineres, que incluem tudo o que o software precisa para funcionar, como bibliotecas, dependências e arquivos de configuração. Isso garante que o aplicativo funcione de maneira consistente em qualquer ambiente.

Em resumo, é uma plataforma que simplifica o processo de desenvolvimento, teste, empacotamento e implantação de aplicativos em contêineres portáteis e leves. O Docker é amplamente utilizado em ambientes de desenvolvimento e produção, permitindo que os desenvolvedores criem aplicativos de forma mais rápida e eficiente.

## Instalação do Docker

Em geral, uma ferramenta muito comum para administrar os contêineres Docker é o Docker Desktop. O Docker Desktop é uma aplicação que fornece uma interface gráfica para gerenciar contêineres Docker, imagens e volumes. Ele é fácil de instalar e configurar, e é uma ótima opção para desenvolvedores que desejam trabalhar com Docker em suas máquinas locais. Ele está disponível para Windows, macOS e Linux.

Por outro lado, existe uma confusão comum entre "Docker Desktop" e "Docker Engine". O Docker Engine refere-se especificamente a um subconjunto dos componentes do Docker Desktop que são gratuitos e de código aberto e podem ser instalados apenas no Linux. O Docker Engine pode criar imagens de contêiner, executar contêineres a partir delas e, em geral, fazer a maioria das coisas que o Docker Desktop pode, mas é apenas para Linux e não fornece todo o polimento da experiência do desenvolvedor que o Docker Desktop fornece.

Para instalar o Docker Desktop em sua máquina, acesse o [site do Docker Desktop](https://docs.docker.com/get-started/get-docker/) e siga as instruções de instalação para o seu sistema operacional. O Docker Desktop é gratuito para uso pessoal e educacional, mas pode ter custos associados para uso comercial em algumas circunstâncias. Consulte a documentação do Docker para obter mais informações sobre preços e licenciamento.

Em sistemas operacionais baseados em Linux, como Ubuntu, você pode instalar o Docker Engine diretamente usando o gerenciador de pacotes. Aqui estão os passos básicos para instalar o Docker Engine no Ubuntu:

=== "Arch Linux"

    ```bash
    sudo pacman -Syu
    sudo pacman -S docker docker-compose
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

=== "Ubuntu"

    ```bash
    sudo apt-get update && sudo apt upgrade
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common \
        gnupg \
        lsb-release

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    sudo systemctl start docker
    sudo systemctl enable docker
    ```

=== "Fedora"

    ```bash
    sudo dnf update -y
    sudo dnf install dnf-plugins-core -y

    sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    sudo systemctl start docker
    sudo systemctl enable docker
    ```

Também, pode ser útil adicionar o usuário atual ao grupo `docker` para evitar a necessidade de usar `sudo` ao executar comandos do Docker. Você pode fazer isso com o seguinte comando:

```bash
sudo usermod -aG docker $USER
```

Depois de adicionar o usuário ao grupo `docker`, você precisará sair e entrar novamente na sessão para que as alterações tenham efeito. Você pode verificar se o usuário foi adicionado corretamente ao grupo `docker` executando o seguinte comando:

```bash
groups $USER
```

Após a instalação, você pode verificar se o Docker está funcionando corretamente executando o seguinte comando:

```bash

docker --version
```

Isso deve exibir a versão do Docker instalada em seu sistema.

Você também pode executar o `Hello World` do docker, com o seguinte comando:

```bash
docker run hello-world
```

Isso deve baixar uma imagem de teste do Docker Hub e executar um contêiner a partir dela. Se tudo estiver funcionando corretamente, você verá uma mensagem de boas-vindas do Docker.

## Os componentes do Docker

Existem três componentes principais no ecossistema Docker:

- **Dockerfile**: Um arquivo de texto contendo instruções (comandos) para construir uma imagem Docker.
- **Imagem Docker**: Um instantâneo de um contêiner, criado a partir de um Dockerfile. As imagens são armazenadas em um registro, como o Docker Hub, e podem ser puxadas ou enviadas para o registro.
- **Contêiner Docker**: Uma instância em execução de uma imagem Docker.

### Estrutura do Dockerfile

Um Dockerfile é um arquivo de texto que contém uma série de instruções para criar uma imagem Docker. Cada instrução no Dockerfile cria uma camada na imagem, e essas camadas são empilhadas para formar a imagem final. Aqui está um exemplo básico de um Dockerfile:

```dockerfile
FROM ubuntu:20.04 # (1)
RUN apt-get update && \
    apt-get install -y nodejs npm
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

1. `FROM`: Especifica a imagem base a ser usada. Neste caso, estamos usando a imagem oficial do Ubuntu 20.04.
2. `RUN`: Executa comandos no contêiner durante a construção da imagem. Aqui, estamos atualizando o sistema e instalando o Node.js e o npm.
3. `WORKDIR`: Define o diretório de trabalho dentro do contêiner. Todos os comandos subsequentes serão executados a partir desse diretório.
4. `COPY`: Copia arquivos do diretório atual (no host) para o contêiner.
5. `EXPOSE`: Informa ao Docker que o contêiner escutará na porta especificada em tempo de execução. Isso não publica a porta, mas é uma boa prática documentar quais portas o contêiner usará.
6. `CMD`: Especifica o comando a ser executado quando o contêiner é iniciado. Neste caso, estamos iniciando o aplicativo Node.js com `npm start`.

O quadro abaixo resume os principais comandos do Dockerfile:

|    Comando    | Descrição                                                                                             |
| :-----------: | :---------------------------------------------------------------------------------------------------- |
|    `FROM`     | Especifica a imagem base a ser usada.                                                                 |
|     `RUN`     | Executa comandos no contêiner durante a construção da imagem.                                         |
|   `WORKDIR`   | Define o diretório de trabalho dentro do contêiner.                                                   |
|    `COPY`     | Copia arquivos do diretório atual (no host) para o contêiner.                                         |
|   `EXPOSE`    | Informa ao Docker que o contêiner escutará na porta especificada.                                     |
|     `CMD`     | Especifica o comando a ser executado quando o contêiner é iniciado.                                   |
| `ENTRYPOINT`  | Define o ponto de entrada do contêiner, permitindo que o contêiner seja executado como um executável. |
|     `ENV`     | Define variáveis de ambiente dentro do contêiner.                                                     |
|   `VOLUME`    | Cria um ponto de montagem para persistência de dados.                                                 |
|     `ARG`     | Define variáveis de construção que podem ser passadas durante a construção da imagem.                 |
|    `LABEL`    | Adiciona metadados à imagem, como autor, versão, etc.                                                 |
|     `ADD`     | Copia arquivos e diretórios do host para o contêiner, com suporte a URLs e extração de arquivos tar.  |
|    `USER`     | Define o usuário a ser usado ao executar o contêiner.                                                 |
| `HEALTHCHECK` | Define um comando para verificar a saúde do contêiner.                                                |
|   `ONBUILD`   | Adiciona instruções que serão executadas quando a imagem for usada como base para outra imagem.       |

A complexidade de um Dockerfile pode variar dependendo do aplicativo que você está criando. À medida que você se familiariza com o Docker, você aprenderá a usar esses comandos de forma mais eficaz.

A ordem dos comandos no Dockerfile é importante, pois cada comando cria uma nova camada na imagem. O Docker tenta otimizar o processo de construção, armazenando em cache as camadas que não mudaram. Portanto, é uma boa prática colocar os comandos que mudam com menos frequência (como a instalação de dependências) antes dos que mudam com mais frequência (como a cópia do código-fonte).

Note que, ao gerar uma imagem, o Docker executa cada comando do Dockerfile em uma nova camada. Isso significa que, se você modificar um comando no Dockerfile, todas as camadas subsequentes também serão reconstruídas, o que pode aumentar o tempo de construção da imagem. Para otimizar isso, é recomendável agrupar comandos relacionados e minimizar o número de camadas criadas.

### A imagem Docker

Uma imagem Docker é um arquivo leve, independente e executável que contém tudo o que é necessário para executar um aplicativo, incluindo o código-fonte, bibliotecas, dependências e arquivos de configuração. As imagens são criadas a partir de um Dockerfile fe podem ser armazenadas em um registro, como o Docker Hub.

As imagens Docker são compostas por várias camadas, cada uma representando uma instrução no Dockerfile. Essas camadas são empilhadas para formar a imagem final. Quando você executa um contêiner a partir de uma imagem, o Docker cria uma camada de leitura e gravação em cima da imagem, permitindo que você faça alterações no contêiner sem afetar a imagem original.

Por isso, as imagens Docker são imutáveis, o que significa que, uma vez criadas, não podem ser alteradas. Se você precisar fazer alterações em uma imagem, precisará criar uma nova imagem a partir do Dockerfile. As alterações que são feitas em um contêiner em execução não afetam a imagem original, e são armazenadas numa camada sobreposta à imagem original.

As imagens Docker são identificadas por um nome e uma tag, que geralmente seguem o formato `nome:tag`. Por exemplo, `meuapp:1.0` refere-se à imagem chamada `meuapp` com a tag `1.0`. Se você não especificar uma tag, o Docker usará a tag `latest` por padrão.

Você pode verificar a listagem de imagens disponíveis em sua máquina local usando o seguinte comando:

```bash
docker image ls
```

Vamos supor que você não tenha ainda nenhuma imagem instalada. Você pode baixar uma imagem do Docker Hub usando o comando `docker pull`. Por exemplo, para baixar a imagem oficial do Nginx, você pode usar o seguinte comando:

```bash
docker pull nginx
```

Isso fará o download da imagem do Nginx e a armazenará localmente. Você pode verificar se a imagem foi baixada com sucesso executando novamente o comando `docker image ls`. Você verá uma saída semelhante a esta:

| REPOSITORY | TAG    | IMAGE ID     | CREATED     | SIZE  |
| ---------- | ------ | ------------ | ----------- | ----- |
| nginx      | latest | 4cad75abc83d | 2 weeks ago | 192MB |

A imagem `nginx` foi baixada e está disponível localmente. Você pode usar essa imagem para criar contêineres Nginx em sua máquina.

### O contêiner Docker

Um contêiner Docker é uma instância em execução de uma imagem Docker. Como já estudamos, os contêineres são isolados uns dos outros e do host, o que significa que eles têm seu próprio sistema de arquivos, rede e processos. Isso permite que você execute vários contêineres em um único host sem conflitos.

Os contêineres são criados a partir de imagens Docker e podem ser iniciados, parados e removidos conforme necessário. Quando você executa um contêiner, o Docker cria uma camada de leitura e gravação em cima da imagem, permitindo que você faça alterações no contêiner sem afetar a imagem original.

Os contêineres são efêmeros por natureza, o que significa que eles podem ser iniciados e parados rapidamente. Quando um contêiner é parado, ele pode ser removido ou reiniciado a partir da imagem original. Isso torna os contêineres ideais para ambientes de desenvolvimento e produção, onde você pode precisar criar e destruir instâncias rapidamente.

Os contêineres podem ser executados em segundo plano (modo "detached") ou em primeiro plano (modo "foreground"). Quando um contêiner é executado em segundo plano, ele continua em execução mesmo depois que o terminal é fechado. Para executar um contêiner em segundo plano, você pode usar a opção `-d` com o comando `docker run`. Por exemplo:

```bash
docker run -d -p 8001:80 nginx
```

Isso executará um contêiner Nginx em segundo plano, mapeando a porta 80 do contêiner para a porta 8001 do host. Você pode acessar o Nginx no seu navegador em `http://localhost:8001`.

Se você quiser ver uma lista dos contêineres em execução, pode usar o seguinte comando:

```bash
docker container ls
```

Isso exibirá uma tabela com informações sobre os contêineres em execução, incluindo o ID do contêiner, o nome da imagem, o status e as portas mapeadas.

Se você quiser ver todos os contêineres, incluindo os parados, pode usar o seguinte comando:

```bash
docker container ls -a
```

Isso exibirá uma tabela com informações sobre todos os contêineres, incluindo os que estão parados.

Para parar um contêiner em execução, você pode usar o seguinte comando:

```bash
docker container stop <container_id>
```

Substitua `<container_id>` pelo ID do contêiner que você deseja parar. Você pode encontrar o ID do contêiner na saída do comando `docker container ls`.

## Comandos do Docker

Abaixo estão alguns comandos essenciais do Docker que você usará com frequência:

- `docker pull <image>`: Baixar uma imagem de um registro, como o Docker Hub.
- `docker build -t <image_name> <path>`: Criar uma imagem a partir de um Dockerfile, onde `<path>` é o diretório que contém o Dockerfile.
- `docker image ls`: Listar todas as imagens disponíveis em sua máquina local.
- `docker run -d -p <host_port>:<container_port> --name <container_name> <image>`: Executar um contêiner a partir de uma imagem, mapeando portas do host para portas do contêiner.
- `docker container ls`: Listar todos os contêineres em execução.
- `docker container stop <container>`: Parar um contêiner em execução.
- `docker container rm <container>`: Remover um contêiner parado.
- `docker image rm <image>`: Remover uma imagem de sua máquina local.
