# Acesso remoto com kubectl

O k3s armazena o arquivo de configuração do cluster em `/etc/rancher/k3s/k3s.yaml`. Por padrão, esse arquivo aponta para `127.0.0.1` — funcionando apenas quando o `kubectl` é executado de dentro do próprio master.

Para operar o cluster a partir da sua **máquina local**, sem precisar de SSH, é necessário:

1. Configurar o servidor de API para aceitar conexões vindas do IP público.
2. Copiar e adaptar o arquivo de configuração para a máquina local.

## Sobre IPs variáveis no Learner Lab

!!!info "Recomendação: prefira o kubectl via SSH no Learner Lab"

    No AWS Learner Lab, **os IPs públicos mudam toda vez que você encerra e reinicia o lab**. Isso significa que qualquer configuração de acesso remoto que dependa do IP público fica desatualizada a cada sessão. Para manter o acesso remoto funcionando, seria necessário repetir os passos abaixo a cada reinício.

    **Na prática, o mais produtivo para este laboratório é executar todos os comandos `kubectl` diretamente no terminal SSH do master** — sem configurar nada na sua máquina local. O acesso remoto é mostrado aqui por razões didáticas: em um ambiente real de produção, onde os IPs são fixos, essa configuração é feita uma única vez.

## Adicionando o IP público ao certificado TLS

O servidor de API do k3s gera um certificado TLS durante a instalação. Esse certificado só aceita conexões endereçadas a `127.0.0.1` e ao IP privado da instância. Para aceitar conexões vindas do IP público, precisamos adicionar esse IP como um **Subject Alternative Name (SAN)** no certificado — o que é feito via flag no serviço do k3s.

No master (via SSH), edite o arquivo de serviço:

```bash
sudo nano /etc/systemd/system/k3s.service
```

Localize a linha que começa com `ExecStart=` e adicione a flag `--tls-san` com o IP público do master. O trecho deve ficar assim:

```ini
ExecStart=/usr/local/bin/k3s \
    server \
    --tls-san <ip-publico-master>
```

!!!warning "Atenção ao formato do arquivo"

    O arquivo usa `\` para continuar o comando na linha seguinte. Adicione `--tls-san` após a última `\` existente, mantendo a indentação com **Tab** (não espaços). Salve com `Ctrl+O`, `Enter` e feche com `Ctrl+X`.

Recarregue o daemon do systemd e reinicie o k3s:

```bash
sudo systemctl daemon-reload
sudo systemctl restart k3s
```

Aguarde alguns segundos e confirme que o cluster voltou ao ar:

```bash
sudo kubectl get nodes
```

## Copiando o kubeconfig para a máquina local

No master, exiba o conteúdo do arquivo de configuração:

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

Copie todo o conteúdo exibido. Na sua máquina local, crie o diretório de configuração (se não existir) e salve o arquivo:

```bash
mkdir -p ~/.kube
# Cole o conteúdo copiado no arquivo abaixo
nano ~/.kube/config
```

Em seguida, substitua o endereço `127.0.0.1` pelo IP público do master:

```bash
sed -i 's/127.0.0.1/<ip-publico-master>/g' ~/.kube/config
```

Teste o acesso remoto:

```bash
kubectl get nodes
```

A saída deve ser idêntica à obtida via SSH no master.

## Atualizando após cada sessão

!!!warning "Repita estes passos a cada reinício do lab"

    Toda vez que o lab for encerrado e reiniciado:

    1. O IP público do master muda.
    2. O certificado TLS do k3s, gerado na instalação, não inclui o novo IP.
    3. O arquivo `~/.kube/config` na sua máquina local tem o IP antigo.

    Para corrigir, você precisa:

    - Atualizar o `--tls-san` no `k3s.service` com o novo IP.
    - Fazer o `daemon-reload` e reiniciar o k3s.
    - Repetir o `sed` no `~/.kube/config` com o novo IP.

    Por esse motivo, **o acesso via SSH continua sendo a forma mais prática** no cotidiano do Learner Lab.
