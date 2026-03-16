# Docker Compose e Redes

## Docker Compose

O Docker Compose é uma ferramenta que permite definir e executar aplicativos Docker compostos por vários contêineres. Com o Docker Compose, você pode usar um arquivo YAML para configurar os serviços, redes e volumes necessários para o seu aplicativo. Isso facilita a orquestração de contêineres e a criação de ambientes de desenvolvimento e produção consistentes.

!!!info "Compose V1 vs V2"

    O Docker Compose possui duas versões principais. A versão original (V1) era um binário separado chamado `docker-compose` (com hífen). A versão atual (V2) é um plugin integrado ao Docker CLI e usa o comando `docker compose` (sem hífen, com espaço). Ao consultar materiais mais antigos na internet, você poderá encontrar o formato `docker-compose up` — o equivalente atual é `docker compose up`. Neste material, utilizaremos sempre a sintaxe do Compose V2.

O formato YAML é uma linguagem de serialização de dados legível por humanos, que é amplamente utilizada para configuração de aplicativos, em especial no contexto de DevOps e infraestrutura como código. O arquivo de configuração do Docker Compose é chamado `docker-compose.yml` e deve estar localizado no diretório raiz do seu projeto.

O Docker Compose é especialmente útil quando você tem um aplicativo que depende de vários serviços, como um banco de dados, um servidor web e um serviço de cache. Com o Docker Compose, você pode definir todos esses serviços em um único arquivo e iniciar todos eles com um único comando.

Todas as configurações que são realizadas na configuração de um `docker-compose` podem ser realizadas diretamente na linha de comando, mas o uso do `docker-compose` é mais prático e organizado.

### Exemplo mínimo: serviço único

Antes de vermos um cenário mais complexo, vamos começar com o exemplo mais simples possível de um arquivo `docker-compose.yml` — um único serviço Nginx:

```yaml title='docker-compose.yml'
services:
  web:
    image: nginx
    ports:
      - '8080:80'
```

Esse arquivo define um serviço chamado `web` que usa a imagem oficial do Nginx e mapeia a porta 80 do contêiner para a porta 8080 do host. Para iniciar o serviço, basta executar:

```bash
docker compose up -d
```

Acesse `http://localhost:8080` no navegador e você verá a página padrão do Nginx. Para parar e remover o serviço:

```bash
docker compose down
```

Esse exemplo é equivalente a executar `docker run -d -p 8080:80 nginx`, mas a vantagem do Compose é que toda a configuração fica declarada em um arquivo versionável. À medida que o projeto cresce e novos serviços são adicionados, basta editar o mesmo arquivo.

### Cenário multi-serviço: PostgreSQL + PgAdmin

Agora, vamos considerar um cenário mais realista. Anteriormente, vimos como executar um contêiner PostgreSQL com persistência de dados usando volumes, e como executar contêineres Nginx com bind mounts.

Considere o seguinte cenário: você executa um contêiner docker com o PostgreSQL e um outro contêiner com a ferramenta `PgAdmin` (1). Contudo, você não quer que o banco de dados PostgreSQL fique exposto na rede, mas sim que o `PgAdmin` tenha acesso a ele. Para isso, você deveria criar uma rede interna entre os dois contêineres. Isso pode ser feito facilmente com o Docker Compose:
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

### Usando arquivos `.env` para credenciais

No exemplo acima, as credenciais do banco de dados e do PgAdmin estão diretamente no arquivo `docker-compose.yml`. Essa abordagem funciona para demonstrações, mas em projetos reais é uma má prática — especialmente quando o `docker-compose.yml` é versionado no Git. Qualquer pessoa com acesso ao repositório teria acesso às senhas.

O Docker Compose permite usar um arquivo `.env` para separar as credenciais da configuração. Crie um arquivo `.env` no mesmo diretório do `docker-compose.yml`:

```env title='.env'
POSTGRES_USER=postgres
POSTGRES_PASSWORD=minha_senha_segura
POSTGRES_DB=sample_db
PGADMIN_EMAIL=admin@admin.com
PGADMIN_PASSWORD=admin_seguro
```

Em seguida, referencie essas variáveis no `docker-compose.yml` usando a sintaxe `${VARIAVEL}`:

```yaml title='docker-compose.yml'
services:
  db:
    image: postgres
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - db-postgres-data:/var/lib/postgresql/data
    networks:
      - internal-network
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - '5050:80'
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_EMAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_PASSWORD}
    networks:
      - internal-network

networks:
  internal-network:
    driver: bridge

volumes:
  db-postgres-data:
    driver: local
```

!!!warning "Atenção"

    Adicione o arquivo `.env` ao seu `.gitignore` para que ele não seja versionado no repositório. Dessa forma, cada desenvolvedor ou ambiente pode ter suas próprias credenciais sem expô-las no repositório.

    ```gitignore title='.gitignore'
    .env
    ```

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

Na maioria das vezes você usará a rede bridge, que permite que os contêineres se comuniquem entre si usando nomes de serviço, o que facilita a configuração e o gerenciamento de aplicativos compostos por vários contêineres.

Para listar as redes disponíveis em sua máquina local, você pode usar o seguinte comando:

```bash
docker network ls
```

Também é possível inspecionar uma rede específica para obter mais informações sobre ela, como os contêineres conectados a ela e as configurações de rede. Para isso, você pode usar o seguinte comando:

```bash
docker network inspect <network_name>
```

### Exercício: inspecionando a rede do Docker Compose

Utilizando o exemplo do Docker Compose com PostgreSQL e PgAdmin que criamos anteriormente, podemos investigar como o Docker configura a rede interna entre os contêineres.

Com os serviços em execução (`docker compose up -d`), execute o seguinte comando para listar as redes criadas:

```bash
docker network ls
```

Você verá uma rede com nome semelhante a `<nome_do_diretório>_internal-network`. Inspecione essa rede para ver os contêineres conectados e seus endereços IP:

```bash
docker network inspect <nome_do_diretório>_internal-network
```

Na saída, procure pela seção `Containers`. Você verá os contêineres `db` e `pgadmin` com seus respectivos endereços IP na rede interna.

Para confirmar que os contêineres conseguem se comunicar pela rede interna usando o nome do serviço, execute um comando `ping` de dentro do contêiner `pgadmin` para o serviço `db`:

```bash
docker compose exec pgadmin ping -c 3 db
```

O Docker resolve automaticamente o nome `db` para o endereço IP do contêiner correspondente, demonstrando na prática o funcionamento do DNS interno do Docker em redes definidas pelo Compose.

## Exercício integrado: aplicação web com banco de dados

Para consolidar os conceitos de persistência, Compose e redes, vamos criar um ambiente completo com uma aplicação web que se conecta a um banco de dados PostgreSQL.

**Passo 1**: Crie um diretório para o projeto:

```bash
mkdir compose-exercicio && cd compose-exercicio
```

**Passo 2**: Crie o arquivo `.env` com as credenciais:

```env title='.env'
POSTGRES_USER=app_user
POSTGRES_PASSWORD=app_senha_123
POSTGRES_DB=app_db
```

**Passo 3**: Crie o arquivo `docker-compose.yml`:

```yaml title='docker-compose.yml'
services:
  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-network

  adminer:
    image: adminer
    ports:
      - '8080:8080'
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  pgdata:
    driver: local
```

Neste exercício, usamos o **Adminer** ao invés do PgAdmin. O Adminer é uma ferramenta mais leve para gerenciamento de banco de dados, que suporta PostgreSQL, MySQL, SQLite e outros. Ele é ideal para fins didáticos por ser mais simples.

**Passo 4**: Inicie os serviços e verifique:

```bash
docker compose up -d
docker compose ps
```

**Passo 5**: Acesse o Adminer em `http://localhost:8080` e conecte-se ao banco de dados usando:

- **Sistema**: PostgreSQL
- **Servidor**: `db` (nome do serviço no Compose)
- **Usuário**: `app_user`
- **Senha**: `app_senha_123`
- **Base de dados**: `app_db`

**Passo 6**: Crie uma tabela e insira dados usando a interface do Adminer.

**Passo 7**: Pare os serviços, remova os contêineres e reinicie:

```bash
docker compose down
docker compose up -d
```

**Passo 8**: Acesse o Adminer novamente e verifique que os dados persistiram, pois o volume `pgdata` mantém os dados entre recriações dos contêineres.

**Passo 9**: Para limpar tudo (incluindo os volumes), use:

```bash
docker compose down -v
```

A opção `-v` remove os volumes junto com os contêineres. Use-a com cuidado, pois os dados armazenados nos volumes serão perdidos.

## Resumo

Neste capítulo, exploramos como organizar e conectar múltiplos contêineres:

- **Docker Compose**: definição declarativa de serviços, volumes e redes em um arquivo YAML versionável.
- **Arquivos `.env`**: separação de credenciais e configurações sensíveis do arquivo de Compose.
- **Redes Docker**: tipos de rede (bridge, host, overlay) e como o Docker resolve nomes de serviço via DNS interno.
- **Integração**: como combinar persistência, redes e Compose para criar ambientes de desenvolvimento completos.

Com esses fundamentos, você tem as ferramentas necessárias para criar ambientes multi-contêiner para desenvolvimento e testes. No próximo capítulo, sobre **Orquestração**, veremos como gerenciar contêineres em escala — quando uma aplicação cresce e precisa de dezenas ou centenas de contêineres distribuídos, ferramentas como Kubernetes se tornam essenciais para automatizar a implantação, o escalonamento e o gerenciamento desses contêineres.
