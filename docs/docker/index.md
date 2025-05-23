# Docker

Criado em 2013, o Docker introduziu o que se tornou o padrão da indústria para contêineres. Ele foi desenvolvido por Solomon Hykes e sua equipe na dotCloud, uma plataforma de PaaS (Platform as a Service). Desde então, o Docker evoluiu para se tornar uma das ferramentas mais populares para desenvolvimento e implantação de aplicativos em contêineres.

Docker é uma plataforma Open Source escrita em Go que ajuda a criação e a administração de ambientes isolados. Ele permite criar, testar e implantar aplicativos rapidamente. O Docker empacota o software em unidades padronizadas chamadas contêineres, que incluem tudo o que o software precisa para funcionar, como bibliotecas, dependências e arquivos de configuração. Isso garante que o aplicativo funcione de maneira consistente em qualquer ambiente.

O Docker trabalha com uma virtualização a nível do sistema operacional, onde o mesmo utiliza de recursos como o kernel do sistema hospedeiro para executar seus contêineres. Diferente do modelo tradicional de Máquinas Virtuais, o Docker não necessita da instalação de um sistema operacional por completo, e sim apenas dos arquivos necessários para a aplicação ser executada.

Em resumo, o Docker simplifica o processo de desenvolvimento, teste, empacotamento, homologação e implantação de aplicativos em contêineres portáteis e leves. O Docker é amplamente utilizado em ambientes de desenvolvimento e produção, permitindo que os desenvolvedores criem aplicativos de forma mais rápida e eficiente, minimizando impactos no processo de desenvolvimento e entrega de software.

Como já discutido anteriormente, existem diversos runtimes de contêineres e até é possível utilizar contêineres sem Docker. Contudo, atualmente o Docker é o runtime de container mais utilizada no mercado, sendo uma tecnologia fundamental no mundo do DevOps.

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

    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    sudo systemctl start docker
    sudo systemctl enable docker
    ```

=== "Debian"

    ```bash
    sudo apt-get update && sudo apt upgrade
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release

    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
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

<!-- ```mermaid
architecture-beta
    group client[Cliente]
    service pull(database)[Docker Pull] in client
    service build(disk)[Docker Build] in client
    service run(disk)[Docker Run] in client

    group host[Docker Host]
    service daemon(database)[Docker Docker Daemon] in host
    service disk3(disk)[Docker Build] in host
    service disk4(disk)[Docker Push] in host

    pull{group}:R -- L:daemon{group}
    build{group}:R -- L:daemon{group}
    run{group}:R -- L:daemon{group}
``` -->

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

## Persistência de dados

Por padrão, o armazenamento dentro de um contêiner Docker é efêmero, o que significa que qualquer alteração ou modificação de dados feita dentro de um contêiner só está disponível enquanto o contêiner estiver em execução. Uma vez que o contêiner é parado e removido, todos os dados associados a ele serão perdidos. Esse armazenamento temporário ou de curta duração é chamado de "sistema de arquivos efêmero do contêiner".

Em linhas gerais, quanto um contêiner inicia, ele usa os arquivos e a configuração fornecidos pela imagem. Cada contêiner é capaz de criar, modificar e excluir arquivos, fazendo isso sem afetar nenhum outro contêiner. Quando o contêiner é excluído, essas alterações de arquivo também são excluídas.

Embora essa natureza efêmera dos contêineres seja ótima para desenvolvimento e testes, ela pode ser um desafio quando você deseja persistir dados entre reinicializações de contêineres. Por exemplo, se você reiniciar um contêiner de banco de dados, pode não querer começar com um banco de dados vazio. Portanto, como persistir arquivos?

Existem duas abordagens principais para persistir dados em contêineres Docker:

1. **Bind mounts**: Os bind mounts permitem que você monte um diretório do host dentro de um contêiner.

2. **Volumes**: Os volumes são a maneira recomendada de persistir dados em contêineres Docker. Eles são armazenados fora do sistema de arquivos do contêiner e podem ser compartilhados entre vários contêineres.

### Bind mounts

Os bind mounts, por sua vez, permitem que você monte um diretório do host dentro de um contêiner. Isso significa que qualquer alteração feita no diretório do host será refletida no contêiner e vice-versa. Para usar um bind mount, você pode usar a opção `-v` com o caminho do diretório do host:

```bash
docker run -d -p 8001:80 -v /path/to/host/directory:/usr/share/nginx/html nginx
```

Isso montará o diretório `/path/to/host/directory` do host no diretório `/usr/share/nginx/html` dentro do contêiner Nginx. Qualquer arquivo criado ou modificado nesse diretório será persistido no diretório do host, mesmo que o contêiner seja removido.

É importante notar que os bind mounts não são gerenciados pelo Docker e podem ser mais difíceis de gerenciar do que os volumes. Além disso, os bind mounts podem apresentar problemas de portabilidade, pois dependem do caminho do diretório no host.

!!!example annotate "Bind mounts na prática"

    Vamos fazer um exemplo de modificação no Nginx. Siga os seguintes passos:


    - [x] Vamos criar uma pasta para o nosso projeto, por exemplo `docker-nginx`
    - [x] Abra esse diretório no Visual Studio Code ou no editor de sua preferência.
    - [x] Crie um sub-diretório com o nome `html` dentro de `docker-nginx`:
    - [x] Crie um arquivo `index.html` dentro do diretório `html` com o seguinte conteúdo:

    ```html
    <!DOCTYPE html>
    <html lang="pt-BR">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Docker Nginx</title>
    </head>
    <body>
        <h1>Bem-vindo ao Docker Nginx!</h1>
        <p>Este é um exemplo de página servida pelo Nginx em um contêiner Docker.</p>
    </body>
    </html>
    ```

    - [x] Agora, execute o seguinte comando no terminal:

    ```bash
    docker run -p 8001:80 -v $(pwd)/html:/usr/share/nginx/html nginx # (1)
    ```

    - [x] Agora, abra o navegador e acesse `http://localhost:8001`. Você verá a página HTML que você criou no diretório `html` do host.

    - [x] Agora, faça uma alteração no arquivo `index.html` e salve. Você verá que a página no navegador será atualizada automaticamente com as alterações feitas no arquivo `index.html` do host.

1. :man_raising_hand: Isso iniciará um contêiner Nginx, mapeando a porta 80 do contêiner para a porta 8001 do host e montando o diretório `html` do host no diretório `/usr/share/nginx/html` dentro do contêiner. Lembre-se que este diretório é onde o Nginx procura os arquivos HTML para servir.

### Volumes

Como citado anteriormente, os volumes são gerenciados pelo Docker e podem ser facilmente criados, removidos e compartilhados. Para criar um volume, você pode usar o seguinte comando:

```bash
docker volume create <volume_name>
```

Para usar um volume ao executar um contêiner, você pode usar a opção `-v`:

```bash
docker run -d -p 8001:80 -v <volume_name>:/usr/share/nginx/html nginx
```

Isso montará o volume `<volume_name>` no diretório `/usr/share/nginx/html` dentro do contêiner Nginx. Qualquer arquivo criado ou modificado nesse diretório será persistido no volume, mesmo que o contêiner seja removido.

Para colocar em prática, vamos usar o exemplo de um contêiner com um servidor de banco de dados PostgreSQL. Como comentado anteriormente, se você reiniciar um contêiner de banco de dados, pode não querer começar com um banco de dados vazio. Dessa forma, o uso de volumes pode ser uma boa solução.

!!!example annotate "Volumes na prática"

    Vamos fazer um exemplo de persistência de dados com o PostgreSQL. Siga os seguintes passos:

    - [x] Primeiro vamos criar um volume com o nome `pgdata`:

    ```bash
    docker volume create pgdata
    ```

    - [x] Agora vamos executar um contêiner PostgreSQL, mapeando o volume `pgdata` para o diretório `/var/lib/postgresql/data` dentro do contêiner:

    ```bash
    docker run -d \
        --name postgres \
        -e POSTGRES_USER=postgres \
        -e POSTGRES_PASSWORD=postgres \
        -p 5432:5432 \
        -v pgdata:/var/lib/postgresql/data \
        postgres # (1)
    ```
    - [x] Agora, você pode acessar o banco de dados PostgreSQL usando um cliente de banco de dados, como o DBeaver ou o pgAdmin, usando as seguintes credenciais:

    ```
    Host: localhost
    Porta: 5432
    Usuário: postgres
    Senha: postgres
    ```

    - [x] Todas as alterações que você fizer no banco de dados serão persistidas no volume `pgdata`, mesmo que o contêiner seja removido.
    - [x] Para parar o contêiner, você pode usar o seguinte comando:

    ```bash
    docker container stop postgres
    ```

1. Note que estamos usando a imagem oficial do PostgreSQL e mapeando o volume `pgdata` para o diretório `/var/lib/postgresql/data` dentro do contêiner. Esse diretório é onde o PostgreSQL armazena os dados do banco de dados. Isso garante que os dados persistam mesmo que o contêiner seja removido. Também estamos mapeando a porta 5432 do contêiner para a porta 5432 do host, permitindo que você acesse o banco de dados a partir do host.

Os volumes são administrados pelo Docker e podem ser facilmente criados, removidos e compartilhados entre contêineres. Você pode listar os volumes disponíveis em sua máquina local usando o seguinte comando:

```bash
docker volume ls
```

Isso exibirá uma tabela com informações sobre os volumes disponíveis, incluindo o nome e o driver do volume.

Uma informação importante é que os dados do volume não ficam armazenados dentro do contêiner, mas sim em um diretório específico no host. O Docker gerencia esse diretório e garante que os dados sejam persistidos mesmo que o contêiner seja removido. Isso significa que você pode remover um contêiner e ainda ter acesso aos dados armazenados no volume.

Para visualizar o local dos dados do volume no host, você pode usar o seguinte comando:

```bash
docker volume inspect <volume_name>
```

Isso exibirá informações detalhadas sobre o volume, incluindo o caminho no host onde os dados estão armazenados. O caminho pode variar dependendo do sistema operacional e da configuração do Docker.

## Docker compose

O Docker Compose é uma ferramenta que permite definir e executar aplicativos Docker compostos por vários contêineres. Com o Docker Compose, você pode usar um arquivo YAML para configurar os serviços, redes e volumes necessários para o seu aplicativo. Isso facilita a orquestração de contêineres e a criação de ambientes de desenvolvimento e produção consistentes.

O formato YAML é uma linguagem de serialização de dados legível por humanos, que é amplamente utilizada para configuração de aplicativos, em especial no contexto de DevOps e infraestrutura como código. O arquivo de configuração do Docker Compose é chamado `docker-compose.yml` e deve estar localizado no diretório raiz do seu projeto.

O Docker Compose é especialmente útil quando você tem um aplicativo que depende de vários serviços, como um banco de dados, um servidor web e um serviço de cache. Com o Docker Compose, você pode definir todos esses serviços em um único arquivo e iniciar todos eles com um único comando.

Todas as configurações que são realizadas na configuração de um `docker-compose` podem ser realizadas diretamente na linha de comando, mas o uso do `docker-compose` é mais prático e organizado. Anteriormente, vimos como executar um contêiner Ngnix com persistência de dados usando `bind mounts`. Também vimos como executar um contêiner PostgreSQL com persistência de dados usando volumes.

Agora, vamos considerar o seguinte cenário: você executa um contêiner docker com o PostgreSQL e um outro contêiner com a ferramenta `PgAdmin` (1). Contudo, você não quer que o banco de dados PostgreSQL fique exposto na rede, mas sim que o `PgAdmin` tenha acesso a ele. Para isso, você deveria criar uma rede interna entre os dois contêineres. Isso pode ser feito facilmente com o Docker Compose, e veremos como fazer isso a seguir.
{ .annotate }

1. O `PgAdmin` é uma ferramenta de gerenciamento e administração de banco de dados PostgreSQL. Ele fornece uma interface gráfica para interagir com o banco de dados, permitindo que você execute consultas SQL, visualize dados e gerencie objetos do banco de dados.

```yaml title='docker-compose.yml'
services:
  db:
    image: postgres
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=sample_db
    volumes:
      - db-postgres-data:/var/lib/postgresql/data
    networks:
      - internal-network
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - '5050:80'
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=admin
    networks:
      - internal-network

networks:
  internal-network:
    driver: bridge

volumes:
  db-postgres-data:
    driver: local
```

Note que o arquivo está dividido em três seções principais: `services`, `networks` e `volumes`. A seção `services` define os serviços que serão executados, a seção `networks` define as redes que serão criadas e a seção `volumes` define os volumes que serão utilizados.

Na seção `services`, temos dois serviços: `db` e `pgadmin`.

- **db** - Esse serviço usa a imagem oficial do PostgreSQL e define algumas variáveis de ambiente para configurar o banco de dados. O banco de dados será criado com o nome `sample_db`, e as credenciais de acesso serão definidas como `postgres` para o usuário e senha. Note que o volume `db-postgres-data` é montado no diretório `/var/lib/postgresql/data` dentro do contêiner, garantindo que os dados do banco de dados sejam persistidos mesmo que o contêiner seja removido. O serviço `db` também está conectado à rede interna `internal-network`, o que significa que ele pode se comunicar com outros serviços na mesma rede.
- **pgadmin** - Esse serviço usa a imagem oficial do PgAdmin e expõe a porta 80 do contêiner na porta 5050 do `host`. As variáveis de ambiente `PGADMIN_DEFAULT_EMAIL` e `PGADMIN_DEFAULT_PASSWORD` são usadas para definir as credenciais de acesso ao PgAdmin. O serviço `pgadmin` também está conectado à rede interna `internal-network`, permitindo que ele se comunique com o serviço `db`.

!!!abstract "Relembrando"

    Os nomes dessas variáveis de ambiente foram definidos na documentação oficial do PgAdmin. Você pode consultar a [documentação oficial do PgAdmin](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html) para mais informações sobre as variáveis de ambiente disponíveis. Cada imagem possui as suas variáveis de ambiente, e é importante consultar a documentação para entender como configurá-las corretamente.

Note que, pela configuração de `networks`, os dois serviços estão conectados à mesma rede interna chamada `internal-network`. Isso significa que eles podem se comunicar entre si, mas não estarão acessíveis diretamente a partir do host. O PgAdmin poderá acessar o banco de dados PostgreSQL usando o nome do serviço `db` como hostname. O próprio Docker irá resolver o nome do serviço para o endereço IP do contêiner correspondente na rede interna, e também gerenciará a atribuição de endereços IP para os contêineres na rede criada.

Para iniciar os serviços definidos no arquivo `docker-compose.yml`, você pode usar o seguinte comando:

```bash
docker compose up # (1) (2)
```

1. Você também pode usar a opção `-d` para executar os serviços em segundo plano (modo "detached").
2. Também você pode definir apenas um serviço específico para iniciar. O comando `docker compose up db` iniciará apenas o serviço `db`.

Depois de iniciado os serviços, você pode acessar o PgAdmin no seu navegador em `http://localhost:5050`. Use as credenciais definidas no arquivo `docker-compose.yml` para fazer login. Depois de autenticado, você pode adicionar uma nova conexão ao banco de dados PostgreSQL usando o nome do serviço `db` como hostname e as credenciais definidas no arquivo `docker-compose.yml`.

Com isso, o seu contêiner `pgadmin` poderá acessar o banco de dados PostgreSQL, que está em outro contêiner, e você poderá gerenciar o banco de dados através da interface gráfica do PgAdmin.

### Comandos do Docker Compose

Abaixo estão alguns comandos essenciais do Docker Compose que você usará com maior frequência:

| Comando                                   | Descrição                                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `docker compose up`                       | Cria e inicia os serviços definidos no arquivo `docker-compose.yml`.                                   |
| `docker compose down`                     | Para e remove os serviços definidos no arquivo `docker-compose.yml`.                                   |
| `docker compose ps`                       | Lista os serviços em execução definidos no arquivo `docker-compose.yml`.                               |
| `docker compose logs`                     | Exibe os logs dos serviços em execução definidos no arquivo `docker-compose.yml`.                      |
| `docker compose build`                    | Constrói as imagens dos serviços definidos no arquivo `docker-compose.yml`.                            |
| `docker compose exec <service> <command>` | Executa um comando em um contêiner em execução de um serviço definido no arquivo `docker-compose.yml`. |
| `docker compose run <service>`            | Executa um comando em um novo contêiner de um serviço definido no arquivo `docker-compose.yml`.        |
| `docker compose config`                   | Valida e exibe a configuração do arquivo `docker-compose.yml`.                                         |
| `docker compose pull`                     | Faz o download das imagens dos serviços definidos no arquivo `docker-compose.yml`.                     |
| `docker compose rm`                       | Remove os contêineres parados dos serviços definidos no arquivo `docker-compose.yml`.                  |
| `docker compose restart`                  | Reinicia os serviços definidos no arquivo `docker-compose.yml`.                                        |
| `docker compose start`                    | Inicia os serviços definidos no arquivo `docker-compose.yml` que estão parados.                        |
| `docker compose stop`                     | Para os serviços definidos no arquivo `docker-compose.yml` que estão em execução.                      |
| `docker compose version`                  | Exibe a versão do Docker Compose instalada.                                                            |
| `docker compose help`                     | Exibe a ajuda para o Docker Compose.                                                                   |
| `docker compose -f <file>`                | Especifica um arquivo de configuração diferente do padrão `docker-compose.yml`.                        |

Existem muitos outros comandos e opções disponíveis no Docker Compose, mas esses são os mais comuns e úteis para começar. Você pode consultar a [documentação oficial do Docker Compose](https://docs.docker.com/compose/) para obter mais informações sobre os comandos e as opções disponíveis.

## Redes Docker

No exemplo que apresentamos, foi criada uma rede interna chamada `internal-network`, que permite que os contêineres se comuniquem entre si. O Docker cria automaticamente uma rede bridge padrão chamada `bridge` quando o Docker é instalado. Essa rede é usada para conectar contêineres em execução no mesmo host.

Em geral, o docker permite a criação de diversos tipos de redes (`drivers`). Aqui, falaremos de três:

- **Bridge**: A rede bridge padrão é criada automaticamente pelo Docker e é usada para conectar contêineres em execução no mesmo host. Essa rede permite que os contêineres se comuniquem entre si usando endereços IP internos.
- **Host**: A rede host conecta o contêiner diretamente à rede do host. Isso significa que o contêiner compartilha o mesmo espaço de rede que o host, permitindo acesso direto às interfaces de rede do host. Essa rede é útil quando você precisa de desempenho máximo e não se importa com o isolamento do contêiner.
- **Overlay**: A rede overlay permite que você conecte contêineres em diferentes hosts Docker. Essa rede é útil para criar aplicativos distribuídos que precisam se comunicar entre diferentes hosts. O Docker Swarm usa redes overlay para conectar serviços em diferentes nós do cluster.

Na maioria das vezes você usará a rede bridge padrão, que permitem que os contêineres se comuniquem entre si usando nomes de serviço, o que facilita a configuração e o gerenciamento de aplicativos compostos por vários contêineres.

Para listar as redes disponíveis em sua máquina local, você pode usar o seguinte comando:

```bash
docker network ls
```

Também é possível inspecionar uma rede específica para obter mais informações sobre ela, como os contêineres conectados a ela e as configurações de rede. Para isso, você pode usar o seguinte comando:

```bash
docker network inspect <network_name>
```
