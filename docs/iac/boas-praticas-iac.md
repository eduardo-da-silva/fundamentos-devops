# Boas práticas de IaC

## Organização de código

- Separe Terraform e Ansible em diretorios distintos.
- Use nomes claros para recursos, variaveis e arquivos.
- Padronize estrutura para facilitar manutenção em equipe.

## Controle de estado

- Não edite state manualmente.
- Em equipe, use backend remoto com lock.
- Versione o código, não o estado local sensível.

## Segurança de credenciais

- Não commite access key, secret key, session token ou chave privada.
- Prefira variáveis de ambiente e arquivos ignorados no Git.
- Revise histórico de commits se houver vazamento acidental.

## Uso de variáveis

- Evite hardcode de região, AMI, tipo de instância e CIDRs.
- Defina defaults seguros quando apropriado.
- Mantenha exemplos em arquivos .example.

## Reutilização

- Crie módulos Terraform para padrões recorrentes.
- Crie roles Ansible para tarefas repetidas.
- Evite duplicação de blocos grandes de código.

## Separação de ambientes

- Separe dev, homolog e prod por variaveis e backends.
- Mude parâmetros, não lógica.
- Use naming/tagging consistente por ambiente.
