# Introdução

DevOps é uma cultura e conjunto de práticas que visa unificar o desenvolvimento de software (Dev) com a operação de sistemas (Ops). Esta abordagem enfatiza a colaboração, automação e integração contínua, permitindo entregas mais rápidas e confiáveis de software.

O termo "DevOps" surgiu em 2009 durante a conferência Agile Conference em Toronto, quando [Patrick Debois](https://x.com/patrickdebois) e [Andrew Shafer](https://x.com/littleidea) discutiram os problemas entre as equipes de desenvolvimento e operações. O movimento ganhou força após a primeira DevOpsDays, realizada na Bélgica no mesmo ano.

A frustração com o modelo tradicional de desenvolvimento, onde existia uma clara separação (e frequentemente conflito) entre as equipes de desenvolvimento e operações, levou à busca por uma abordagem mais integrada. As equipes de desenvolvimento queriam implementar mudanças rapidamente, enquanto as equipes de operações priorizavam a estabilidade do sistema.

Esta divisão resultava em:

- Longos ciclos de desenvolvimento
- Problemas frequentes em produção
- Comunicação deficiente entre equipes
- Resistência à mudança

O DevOps surgiu como resposta a esses desafios, propondo uma cultura de colaboração e integração entre as equipes.

## Fundamentos de DevOps

Dentro de uma organização que adota DevOps, as equipes de desenvolvimento e operações trabalham juntas para alcançar os objetivos comuns de entrega rápida e confiável de software. Isso envolve a implementação de práticas ágeis, automação de processos e monitoramento contínuo do desempenho do software.

De forma resumida, os princípios fundamentais do DevOps incluem:

- **Cultura de colaboração entre equipes**: A colaboração entre as equipes de desenvolvimento e operações é essencial. Por exemplo, em um projeto de desenvolvimento de uma aplicação web, as equipes podem realizar reuniões diárias para discutir o progresso, identificar obstáculos e planejar as próximas etapas juntos.

- **Automação de processos**: A automação reduz erros humanos e acelera o ciclo de desenvolvimento. Um exemplo prático é a utilização de pipelines de integração contínua (CI) e entrega contínua (CD) para automatizar testes e implantações. Ferramentas como Jenkins, GitLab CI/CD e GitHub Actions são amplamente utilizadas para esse fim.

- **Medição contínua de resultados**: Medir e monitorar o desempenho do software e dos processos é crucial para identificar áreas de melhoria. Por exemplo, a equipe pode usar ferramentas como Prometheus e Grafana para monitorar a performance da aplicação em produção e gerar alertas em caso de problemas.

- **Compartilhamento de conhecimento**: Promover uma cultura de aprendizado contínuo e compartilhamento de conhecimento entre as equipes. Isso pode ser feito através de sessões de retrospectiva, onde as equipes discutem o que funcionou bem e o que pode ser melhorado, ou através de documentações que detalham as melhores práticas e lições aprendidas.

- **Segurança integrada**: Incorporar práticas de segurança desde o início do ciclo de desenvolvimento. Isso é conhecido como DevSecOps. Por exemplo, realizar análises de vulnerabilidades e testes de segurança automatizados durante o processo de CI/CD para garantir que o código esteja seguro antes de ser implantado em produção.

Esses princípios ajudam a criar um ambiente onde as mudanças podem ser implementadas de forma rápida e segura, resultando em entregas mais frequentes e de maior qualidade. Nos dias atuais, a adoção de DevOps é essencial para as empresas que buscam se manter competitivas em um mercado cada vez mais dinâmico e exigente.

Isso porque a capacidade de responder rapidamente às demandas dos clientes e às mudanças no mercado é um diferencial competitivo importante. Ao adotar os conceitos e técnicas de DevOps, as empresas têm maior flexibilidade para lançar novas funcionalidades, corrigir bugs e ajustar o software de acordo com as necessidades do negócio.

Dessa forma, se bem empregada, a implementação bem-sucedida do DevOps resulta em:

- **Entregas mais rápidas**: Com a automação de processos e a integração contínua, as equipes podem lançar novas funcionalidades e correções de bugs com maior frequência. Por exemplo, uma equipe de desenvolvimento pode usar pipelines de CI/CD para automatizar a construção, teste e implantação de uma aplicação, reduzindo o tempo entre a escrita do código e sua disponibilização em produção.

- **Maior qualidade de software**: Através de testes automatizados e monitoramento contínuo, é possível identificar e corrigir problemas antes que eles afetem os usuários finais. Por exemplo, a utilização de testes unitários, testes de integração e testes de aceitação automatizados garante que o código esteja funcionando conforme o esperado em diferentes cenários.

- **Melhor resposta a problemas**: Com a implementação de práticas de monitoramento e logging, as equipes podem detectar e responder a incidentes mais rapidamente. Por exemplo, ao usar ferramentas como ELK Stack (Elasticsearch, Logstash e Kibana) para centralizar e analisar logs, a equipe pode identificar a causa raiz de um problema e aplicar correções de forma ágil.

- **Maior satisfação do cliente**: A capacidade de entregar novas funcionalidades e melhorias de forma contínua e confiável aumenta a satisfação do cliente. Por exemplo, uma empresa de e-commerce que adota DevOps pode lançar novas funcionalidades de forma incremental, respondendo rapidamente ao feedback dos usuários e melhorando a experiência de compra.

Esses benefícios demonstram como a adoção de práticas DevOps pode transformar a forma como as equipes de desenvolvimento e operações trabalham, resultando em um ciclo de desenvolvimento mais eficiente e produtos de maior qualidade.

## Versionamento de Código

O versionamento de código é uma prática essencial no desenvolvimento de software que consiste em controlar e gerenciar as alterações feitas no código-fonte ao longo do tempo. Isso permite que as equipes de desenvolvimento trabalhem de forma colaborativa, rastreiem as mudanças realizadas e revertam para versões anteriores se necessário.

No contexto do DevOps, o versionamento de código é fundamental por várias razões:

- **Colaboração eficiente**: Ferramentas de versionamento, como Git, permitem que múltiplos desenvolvedores trabalhem simultaneamente no mesmo projeto sem conflitos. Por exemplo, em um projeto de desenvolvimento de uma aplicação web, diferentes desenvolvedores podem trabalhar em funcionalidades distintas e integrar suas mudanças de forma coordenada.

- **Histórico de mudanças**: Manter um histórico detalhado das alterações facilita a identificação de quando e por quem uma mudança foi feita. Isso é crucial para auditorias e para entender a evolução do projeto. Por exemplo, se um bug for introduzido, a equipe pode revisar o histórico de commits para encontrar a origem do problema.

- **Reversão de alterações**: Caso uma mudança cause problemas, é possível reverter para uma versão anterior do código de forma rápida e segura. Isso minimiza o impacto de erros em produção. Por exemplo, se uma nova funcionalidade causar falhas, a equipe pode reverter para a versão estável anterior enquanto trabalha na correção.

- **Integração contínua**: O versionamento de código é a base para a integração contínua (CI), onde o código é frequentemente integrado e testado automaticamente. Isso garante que as mudanças sejam verificadas continuamente, reduzindo o risco de problemas em produção. Ferramentas como Jenkins e GitHub Actions são usadas para configurar pipelines de CI que automatizam esses processos.

- **Entrega contínua**: Com o versionamento de código, é possível automatizar a entrega contínua (CD), onde as mudanças são automaticamente implantadas em ambientes de produção após passarem por testes rigorosos. Isso acelera o ciclo de desenvolvimento e entrega de software. Por exemplo, uma equipe pode configurar um pipeline de CD que implanta automaticamente novas versões da aplicação em um ambiente de produção após a aprovação dos testes.

Esses benefícios mostram como o versionamento de código é uma prática indispensável para a implementação bem-sucedida do DevOps, promovendo um desenvolvimento mais ágil, colaborativo e seguro.

### Práticas de versionamento

??? note "Nota importante sobre Git"

    Aqui, estou considerando que você já conheça o Git. Caso ainda não tenha familiaridade com a ferramenta, sugiro buscar algum material para estudo.

    Para mais detalhes, veja esse [material ](git-basico.md), com algumas dicas iniciais de Git.


    [instant loading]: setup/setting-up-navigation.md/#instant-loading
    [RxJS Observable]: https://rxjs.dev/api/index/class/Observable

Existem várias práticas e convenções que podem ser adotadas para facilitar o versionamento de código e a colaboração entre desenvolvedores. Alguns dos conceitos mais comuns incluem:

- **Branches**: Dividir o código em branches (ramificações) separadas para desenvolver funcionalidades ou correções de bugs isoladamente. Por exemplo, uma equipe pode criar um branch `feature/nova-funcionalidade` para implementar uma nova funcionalidade sem interferir no código principal.

- **Merges**: Integrar as mudanças de um branch para outro, geralmente usando pull requests. Por exemplo, após concluir o desenvolvimento de uma funcionalidade em um branch de feature, a equipe pode abrir um pull request para mesclar as mudanças no branch principal.

- **Conventional Commits**: Adotar uma convenção de mensagens de commit padronizada para facilitar a compreensão do histórico de mudanças. Por exemplo, prefixar as mensagens de commit com palavras-chave como `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `test` ou `ci` para indicar o tipo de alteração realizada.

- **GitFlow**: Seguir um modelo de branching como o GitFlow, que define um fluxo de trabalho com branches específicos para features, releases, hotfixes e versões. Por exemplo, o GitFlow propõe o uso de branches `feature/`, `release/`, `hotfix/` e `develop` para organizar o desenvolvimento de forma estruturada.

Essas práticas ajudam a manter o código organizado, facilitam a colaboração entre desenvolvedores e garantem a consistência do histórico de mudanças ao longo do tempo.

### Práticas de SemVer

O [SemVer](https://semver.org/) (Semantic Versioning) é uma convenção de versionamento que define um padrão para atribuir versões a um software com base nas mudanças realizadas. O SemVer é composto por três números separados por pontos: `MAJOR.MINOR.PATCH`.

- `MAJOR`: incrementado quando são feitas alterações incompatíveis com versões anteriores.
- `MINOR`: incrementado quando são adicionadas funcionalidades de forma retrocompatível.
- `PATCH`: incrementado quando são feitas correções de bugs de forma retrocompatível.

Um exemplo de uso do SemVer seria:

- `1.2.3`: versão `1` (major), `2` (minor) e `3` (patch).

Essa convenção facilita a comunicação entre desenvolvedores, usuários e ferramentas, permitindo que todos entendam o impacto das mudanças em uma nova versão do software. Por exemplo, ao seguir o SemVer, os usuários podem saber se uma atualização contém apenas correções de bugs (`PATCH`), novas funcionalidades (`MINOR`) ou mudanças incompatíveis (`MAJOR`). No caso de uma API pública, por exemplo, sabe-se que houver uma alteração no `MAJOR`, a API foi modificada de forma incompatível com a versão anterior.

## Conventional Commits

O Conventional Commits é uma convenção de mensagens de commit padronizada que facilita a compreensão do histórico de mudanças em um repositório de código. Essa convenção define um formato consistente para as mensagens de commit, permitindo que desenvolvedores e ferramentas identifiquem rapidamente o tipo de alteração realizada.

Um bom local de consulta é o [site oficial do Conventional Commits](https://www.conventionalcommits.org/), que fornece informações detalhadas sobre a convenção, exemplos de mensagens de commit e recomendações de uso. Ele é mantido pela comunidade e é uma referência útil para quem deseja adotar a convenção em seus projetos. O projeto foi inspirado em outras convenções de mensagens de commit, em especial o Angular Commit Message Guidelines.

Em linhas gerais, as mensagens de commit seguem um padrão predefinido, que inclui um prefixo indicando o tipo de alteração realizada, seguido de um título descritivo. Alguns dos prefixos mais comuns incluem:

- `feat`: Para novas funcionalidades ou melhorias
- `fix`: Para correções de bugs
- `chore`: Para tarefas de manutenção ou refatoração
- `docs`: Para alterações na documentação
- `style`: Para alterações de estilo ou formatação
- `refactor`: Para refatorações de código
- `test`: Para adições ou alterações em testes
- `ci`: Para alterações em configurações de CI/CD

Por exemplo, uma mensagem de commit seguindo a convenção Conventional Commits pode ser:

```
feat: adiciona funcionalidade de login
```

Essa mensagem indica que o commit adiciona uma nova funcionalidade de login ao projeto. Ao seguir essa convenção, os desenvolvedores podem facilmente identificar o propósito de cada commit e entender as mudanças realizadas ao longo do tempo.

Além disso, ferramentas de automação, como linters e geradores de changelog, podem ser configuradas para analisar as mensagens de commit e gerar informações úteis, como logs de alterações e notas de versão automaticamente.

A adoção do Conventional Commits é uma prática recomendada para equipes que buscam manter um histórico de mudanças claro e organizado, facilitando a colaboração e a revisão de código.

### Ferramentas para auxiliar no cumprimento da convenção

Para facilitar o uso do Conventional Commits, você pode instalar extensões no Visual Studio Code que fornecem suporte para a convenção. Essas extensões ajudam a formatar as mensagens de commit de acordo com a convenção, garantindo consistência e facilitando a leitura do histórico de mudanças.

Uma extensão popular para o Visual Studio Code é o [Conventional Commits](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits), que fornece recursos como autocompletar, validação e formatação das mensagens de commit de acordo com a convenção.

Caso você não tenha interesse em fazer os commits dentro do VS Code, você pode utilizar ferramentas como o [Commitizen](https://commitizen.github.io/cz-cli/) para auxiliar na criação de mensagens de commit seguindo a convenção. O Commitizen é uma ferramenta de linha de comando que fornece um assistente interativo para criar mensagens de commit padronizadas.

Ainda, é possível utilizar ferramentas como o Commitlint e o Husky para validar as mensagens de commit automaticamente durante o processo de commit, garantindo que todas as mensagens sigam a convenção corretamente.
