# Docker

Criado em 2013, o Docker introduziu o que se tornou o padrão da indústria para contêineres. Ele foi desenvolvido por Solomon Hykes e sua equipe na dotCloud, uma plataforma de PaaS (Platform as a Service). Desde então, o Docker evoluiu para se tornar uma das ferramentas mais populares para desenvolvimento e implantação de aplicativos em contêineres.

Docker é uma plataforma Open Source escrita em Go que ajuda a criação e a administração de ambientes isolados. Ele permite criar, testar e implantar aplicativos rapidamente. O Docker empacota o software em unidades padronizadas chamadas contêineres, que incluem tudo o que o software precisa para funcionar, como bibliotecas, dependências e arquivos de configuração. Isso garante que o aplicativo funcione de maneira consistente em qualquer ambiente.

O Docker trabalha com uma virtualização a nível do sistema operacional, onde o mesmo utiliza de recursos como o kernel do sistema hospedeiro para executar seus contêineres. Diferente do modelo tradicional de Máquinas Virtuais, o Docker não necessita da instalação de um sistema operacional por completo, e sim apenas dos arquivos necessários para a aplicação ser executada.

Em resumo, o Docker simplifica o processo de desenvolvimento, teste, empacotamento, homologação e implantação de aplicativos em contêineres portáteis e leves. O Docker é amplamente utilizado em ambientes de desenvolvimento e produção, permitindo que os desenvolvedores criem aplicativos de forma mais rápida e eficiente, minimizando impactos no processo de desenvolvimento e entrega de software.

Como já discutido anteriormente, existem diversos runtimes de contêineres e até é possível utilizar contêineres sem Docker. Contudo, atualmente o Docker é o runtime de container mais utilizado no mercado, sendo uma tecnologia fundamental no mundo do DevOps.

No capítulo anterior, estudamos os fundamentos de isolamento (namespaces) e controle de recursos (cgroups) que tornam os contêineres possíveis. O Docker abstrai toda essa complexidade, oferecendo uma interface de linha de comando e um conjunto de ferramentas para criar, distribuir e executar contêineres de forma prática.

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

Existem três componentes principais no ecossistema Docker:

- **Dockerfile**: Um arquivo de texto contendo instruções (comandos) para construir uma imagem Docker.
- **Imagem Docker**: Um instantâneo de um contêiner, criado a partir de um Dockerfile. As imagens são armazenadas em um registro, como o Docker Hub, e podem ser puxadas ou enviadas para o registro.
- **Contêiner Docker**: Uma instância em execução de uma imagem Docker.

### Estrutura do Dockerfile

Um Dockerfile é um arquivo de texto que contém uma série de instruções para criar uma imagem Docker. Cada instrução no Dockerfile cria uma camada na imagem, e essas camadas são empilhadas para formar a imagem final. Aqui está um exemplo básico de um Dockerfile:

```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y nodejs npm
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

Vamos analisar cada instrução utilizada nesse exemplo:

- `FROM ubuntu:20.04`: Especifica a imagem base a ser usada. Neste caso, estamos usando a imagem oficial do Ubuntu 20.04.
- `RUN apt-get update && apt-get install -y nodejs npm`: Executa comandos no contêiner durante a construção da imagem. Aqui, estamos atualizando o sistema e instalando o Node.js e o npm.
- `WORKDIR /app`: Define o diretório de trabalho dentro do contêiner. Todos os comandos subsequentes serão executados a partir desse diretório.
- `COPY . .`: Copia os arquivos do diretório atual (no host) para o diretório de trabalho do contêiner.
- `RUN npm install`: Instala as dependências do projeto Node.js.
- `EXPOSE 3000`: Informa ao Docker que o contêiner escutará na porta 3000 em tempo de execução. Isso não publica a porta, mas é uma boa prática documentar quais portas o contêiner usará.
- `CMD ["npm", "start"]`: Especifica o comando a ser executado quando o contêiner é iniciado.

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

### Exercício: construindo sua primeira imagem

Vamos criar um projeto mínimo para praticar a construção de uma imagem Docker. Neste exercício, criaremos uma aplicação Node.js simples e a empacotaremos em um contêiner.

**Passo 1**: Crie um diretório para o projeto e inicialize um projeto Node.js:

```bash
mkdir meu-app-docker && cd meu-app-docker
npm init -y
npm install express
```

**Passo 2**: Crie o arquivo `index.js` com uma aplicação web mínima:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello DevOps! Esta aplicação está rodando em um contêiner Docker.');
});

app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

**Passo 3**: Crie o `Dockerfile` na raiz do projeto:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

Note que, seguindo a boa prática mencionada anteriormente, copiamos primeiro os arquivos `package*.json` e instalamos as dependências _antes_ de copiar o restante do código-fonte. Dessa forma, se apenas o código-fonte mudar, a camada de instalação de dependências será reutilizada do cache.

**Passo 4**: Construa a imagem e execute o contêiner:

```bash
docker build -t meu-app:1.0 .
docker run -d -p 3000:3000 --name meu-app meu-app:1.0
```

**Passo 5**: Acesse `http://localhost:3000` no navegador. Você verá a mensagem "Hello DevOps!".

Para encerrar o contêiner após o teste:

```bash
docker container stop meu-app
docker container rm meu-app
```

### A imagem Docker

Uma imagem Docker é um arquivo leve, independente e executável que contém tudo o que é necessário para executar um aplicativo, incluindo o código-fonte, bibliotecas, dependências e arquivos de configuração. As imagens são criadas a partir de um Dockerfile e podem ser armazenadas em um registro, como o Docker Hub.

As imagens Docker são compostas por várias camadas, cada uma representando uma instrução no Dockerfile. Essas camadas são empilhadas para formar a imagem final. Quando você executa um contêiner a partir de uma imagem, o Docker cria uma camada de leitura e gravação em cima da imagem, permitindo que você faça alterações no contêiner sem afetar a imagem original.

Por isso, as imagens Docker são imutáveis, o que significa que, uma vez criadas, não podem ser alteradas. Se você precisar fazer alterações em uma imagem, precisará criar uma nova imagem a partir do Dockerfile. As alterações que são feitas em um contêiner em execução não afetam a imagem original, e são armazenadas numa camada sobreposta à imagem original.

As imagens Docker são identificadas por um nome e uma tag, que geralmente seguem o formato `nome:tag`. Por exemplo, `meuapp:1.0` refere-se à imagem chamada `meuapp` com a tag `1.0`. Se você não especificar uma tag, o Docker usará a tag `latest` por padrão.

Você pode verificar a listagem de imagens disponíveis em sua máquina local usando o seguinte comando:

```bash
docker image ls
```

Vamos supor que você não tenha ainda nenhuma imagem baixada. Você pode baixar uma imagem do Docker Hub usando o comando `docker pull`. Por exemplo, para baixar a imagem oficial do Nginx, você pode usar o seguinte comando:

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

## Docker Hub e registros de imagens

Até agora, usamos imagens como `nginx`, `postgres` e `node` sem nos preocupar de onde elas vieram. Essas imagens são baixadas de um **registro de imagens** (_container registry_) — um repositório centralizado onde imagens Docker são armazenadas e distribuídas.

O registro mais conhecido é o [Docker Hub](https://hub.docker.com/), que funciona como o "GitHub das imagens Docker". Nele, você encontra:

- **Imagens oficiais**: mantidas pela Docker ou pelos mantenedores do projeto (ex: `nginx`, `postgres`, `node`, `python`). São identificadas pelo selo "Docker Official Image".
- **Imagens da comunidade**: publicadas por desenvolvedores e organizações, no formato `usuario/imagem` (ex: `dpage/pgadmin4`).
- **Imagens privadas**: repositórios com acesso restrito, disponíveis em planos pagos.

Quando você executa `docker pull nginx`, o Docker busca a imagem no Docker Hub por padrão. Você também pode especificar outro registro:

```bash
# Puxar do Docker Hub (padrão)
docker pull nginx

# Puxar de um registro alternativo (ex: GitHub Container Registry)
docker pull ghcr.io/usuario/minha-imagem:1.0
```

Para enviar suas próprias imagens ao Docker Hub, você precisa:

1. Criar uma conta em [hub.docker.com](https://hub.docker.com/)
2. Fazer login via terminal: `docker login`
3. Nomear a imagem com seu usuário: `docker tag meu-app:1.0 seuusuario/meu-app:1.0`
4. Enviar: `docker push seuusuario/meu-app:1.0`

Existem outros registros populares além do Docker Hub, como o **GitHub Container Registry** (ghcr.io), o **Amazon ECR**, o **Google Artifact Registry** e o **Azure Container Registry**. Em ambientes corporativos, é comum usar registros privados para armazenar imagens internas da organização.

## Boas práticas para Dockerfile

À medida que você cria mais imagens Docker, alguns padrões se destacam como boas práticas que melhoram a segurança, o tamanho e o desempenho das suas imagens:

### Use imagens base mínimas

Prefira imagens baseadas em Alpine ou variantes `-slim` em vez de imagens completas. Elas são significativamente menores:

```dockerfile
# Ruim: imagem completa (~1GB)
FROM node:20

# Bom: imagem Alpine (~150MB)
FROM node:20-alpine
```

### Copie dependências antes do código

Já vimos essa prática no exercício anterior, mas vale reforçar: copie os arquivos de dependência (`package.json`, `requirements.txt`, etc.) e instale-as **antes** de copiar o código-fonte. Isso maximiza o uso do cache de camadas:

```dockerfile
COPY package*.json ./
RUN npm install
# Só depois copie o código — se ele mudar, as dependências não serão reinstaladas
COPY . .
```

### Combine comandos RUN

Cada instrução `RUN` cria uma nova camada na imagem. Combine comandos relacionados para reduzir o número de camadas e o tamanho final:

```dockerfile
# Ruim: 3 camadas
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# Bom: 1 camada
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### Use `.dockerignore`

Assim como o `.gitignore` para o Git, o arquivo `.dockerignore` impede que arquivos desnecessários sejam copiados para a imagem durante o `COPY . .`:

```dockerignore title='.dockerignore'
node_modules
.git
.env
*.md
```

### Não execute como root

Por padrão, processos dentro do contêiner rodam como `root`. Para maior segurança, crie um usuário não-privilegiado:

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

## Interagindo com contêineres: `docker exec`

Muitas vezes, é necessário executar comandos dentro de um contêiner que já está em execução — seja para depurar problemas, inspecionar arquivos ou executar tarefas administrativas. O comando `docker exec` permite exatamente isso.

A sintaxe básica é:

```bash
docker exec [opções] <container> <comando>
```

As opções mais comuns são `-i` (interativo) e `-t` (aloca um terminal), geralmente combinadas como `-it` para obter um terminal interativo:

```bash
# Abrir um shell dentro de um contêiner em execução
docker exec -it meu-container /bin/bash

# Em contêineres Alpine (que não têm bash):
docker exec -it meu-container /bin/sh

# Executar um comando único (sem modo interativo):
docker exec meu-container ls /app

# Ver variáveis de ambiente do contêiner:
docker exec meu-container env
```

### Exercício: explorando um contêiner por dentro

Vamos praticar o uso do `docker exec` para entender como é o ambiente interno de um contêiner.

**Passo 1**: Inicie um contêiner Nginx:

```bash
docker run -d --name nginx-teste nginx
```

**Passo 2**: Abra um shell interativo dentro do contêiner:

```bash
docker exec -it nginx-teste /bin/bash
```

**Passo 3**: Dentro do contêiner, explore o ambiente:

```bash
# Veja o sistema operacional base
cat /etc/os-release

# Liste os processos em execução
ps aux

# Veja os arquivos de configuração do Nginx
ls /etc/nginx/

# Veja o conteúdo servido pelo Nginx
cat /usr/share/nginx/html/index.html

# Saia do contêiner
exit
```

Note que o contêiner tem seu próprio sistema de arquivos, seus próprios processos e sua própria configuração — exatamente o isolamento que estudamos no capítulo de Contêineres.

**Passo 4**: Limpe o contêiner de teste:

```bash
docker container stop nginx-teste
docker container rm nginx-teste
```

Nos próximos capítulos, veremos como o `docker exec` é usado em conjunto com o Docker Compose (`docker compose exec`) para interagir com serviços específicos em ambientes multi-contêiner.
