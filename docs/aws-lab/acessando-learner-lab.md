# Acessando o AWS Learner Lab

O AWS Learner Lab é acessado por meio da plataforma do seu curso (normalmente o LMS, como Canvas ou Brightspace). Localize o módulo do laboratório e clique para abri-lo — você verá uma interface dedicada com alguns elementos principais.

## A interface do Learner Lab

A tela principal do Learner Lab apresenta:

- **Botões Start Lab / End Lab**: controlam o ciclo de vida do seu ambiente AWS.
- **Um indicador colorido** (círculo ou ícone): mostra o estado atual do laboratório.
- **Botão AWS**: abre o console da AWS em uma nova aba quando o lab está ativo.
- **Ícone AWS Details**: exibe informações da sessão, incluindo a chave de acesso SSH.

## Iniciando o laboratório

Clique em **Start Lab** para provisionar sua conta AWS temporária. O processo leva de 1 a 2 minutos. Acompanhe o indicador colorido:

| Cor      | Significado                                                              |
| -------- | ------------------------------------------------------------------------ |
| Vermelho | Laboratório parado. Nenhum recurso AWS disponível.                       |
| Amarelo  | Laboratório iniciando ou encerrando. Aguarde — não clique em nada ainda. |
| Verde    | Laboratório ativo. O console AWS está disponível.                        |

Somente quando o indicador estiver **verde** você pode clicar em **AWS** para abrir o console. Clicar antes pode resultar em erro de autenticação.

## Baixando a chave PEM

!!!warning "Faça isso antes de criar qualquer instância EC2"

    A chave PEM é necessária para conectar via SSH nas máquinas que vamos criar. Ela **só pode ser baixada neste momento** — não será possível recuperá-la depois. Se pular esse passo, terá que recriar as instâncias.

Com o lab ativo (indicador verde), clique no ícone **AWS Details** — ele fica no mesmo menu dos botões Start Lab / End Lab. Na seção **SSH key**, clique em **Download PEM** e salve o arquivo como `labsuser.pem` em um local de fácil acesso no seu computador.

Essa chave é usada para autenticação ao conectar nas instâncias EC2 que criaremos nas próximas etapas.

## Abrindo o console AWS

Com o lab ativo, clique no botão **AWS**. Isso abre o Console de Gerenciamento da AWS em uma nova aba, já autenticado com as credenciais temporárias do seu laboratório.

!!!info "Credenciais temporárias"

    A conta AWS do Learner Lab tem permissões pré-configuradas para os serviços utilizados nos laboratórios do curso. Algumas operações avançadas (como criação de usuários IAM ou acesso a certos serviços) podem estar bloqueadas por política. Isso é esperado — o foco é nos serviços de computação e rede.

## Encerrando o laboratório

Ao terminar o trabalho, clique em **End Lab**. Isso para todas as instâncias EC2 em execução e suspende o consumo de créditos.

!!!danger "Não esqueça de encerrar o lab"

    Deixar instâncias EC2 rodando sem necessidade consome os créditos do laboratório. Os créditos são limitados e compartilhados com os demais laboratórios do curso. Sempre clique em **End Lab** ao terminar a sessão.

!!!note "Suas instâncias não são destruídas automaticamente"

    Encerrar o lab via End Lab **para** as instâncias, mas não as exclui. Na próxima sessão, ao clicar em Start Lab, as instâncias voltarão a existir — mas com novos IPs públicos. Veremos como lidar com isso nas próximas seções.
