# Criando o projeto

Neste guia, vamos criar um projeto simples em Python para demonstrar a integração contínua com o GitHub Actions. O projeto consistirá em um script Python que realiza a soma de dois números e exibe o resultado.

## Passo a passo

- [x] Crie um diretório para o projeto.

Crie um diretório para o projeto em sua máquina local e abra-o com o Visual Studio Code ou outro editor de sua preferência.

- [x] Inicialize um repositório Git

Dentro do diretório do projeto, inicialize um repositório Git com o comando:

```bash
git init
```

- [x] Crie um ambiente virtual

Crie um ambiente virtual para o projeto com o comando:

```bash
python -m venv .venv
```

Em seguida, ative o ambiente virtual:

=== "Linux e macOS"

    ```bash
    source .venv/bin/activate
    ```

=== "Windows"

    ```powershell
    .venv\Scripts\activate
    ```

- [x] Configure o .gitignore

Crie um arquivo `.gitignore` na raiz do projeto e adicione o seguinte conteúdo para ignorar o ambiente virtual e os arquivos `.pyc`:

```plaintext title=".gitignore"
.venv
*.pyc
__pycache__/
.pytest_cache/
```

- [x] Instale o `pytest`

Instale o `pytest` para realizar os testes unitários do projeto:

```bash
pip install pytest
```

- [x] Crie um arquivo de testes

Crie um arquivo Python chamado `test_app.py` com o seguinte conteúdo:

```py title="test_app.py"
from app import sumF


def test_sum():
    assert sumF(1, 2) == 3
    assert sumF(2, 2) == 4
    assert sumF(3, 2) == 5

```

Note que estamos escrevendo os **testes antes da implementação**. Essa abordagem é conhecida como **TDD** (_Test-Driven Development_): primeiro definimos o comportamento esperado através de testes, e só depois escrevemos o código que os satisfaz. O ciclo do TDD segue três passos: (1) escrever um teste que falha, (2) implementar o código mínimo para fazê-lo passar, e (3) refatorar se necessário.

Se rodarmos os testes agora, eles devem falhar, pois ainda não implementamos a função `sumF`. Para testar, execute o comando:

```bash
pytest
```

- [x] Crie um arquivo Python

Crie um arquivo Python chamado `app.py` com o seguinte conteúdo:

```python title="app.py"
def sumF(a, b):
    return a + b


if __name__ == "__main__":
    print(sumF(10, 10))
```

- [x] Execute os testes

Execute os testes novamente com o comando `pytest`. Eles devem passar agora, pois a função `sumF` foi implementada e está correta.

- [x] Crie um arquivo `requirements.txt`

Crie um arquivo `requirements.txt` com as dependências do projeto:

```bash
pip freeze > requirements.txt
```

## Publicando no GitHub

Neste momento, precisamos publicar o projeto no GitHub para que possamos configurar o GitHub Actions na próxima etapa.

**Passo 1**: Crie um novo repositório no GitHub (pela interface web ou pelo CLI `gh`). Não inicialize com README ou `.gitignore` — já temos esses arquivos localmente.

**Passo 2**: Adicione o repositório remoto e faça o push:

```bash
git add .
git commit -m "Projeto inicial com testes"
git branch -M main
git remote add origin https://github.com/seu-usuario/ci-sum-python.git
git push -u origin main
```

Substitua `seu-usuario` pelo seu nome de usuário no GitHub e `ci-sum-python` pelo nome que você escolheu para o repositório.

Após o push, verifique na interface do GitHub que todos os arquivos (`app.py`, `test_app.py`, `requirements.txt`, `.gitignore`) estão presentes no repositório. Na próxima página, vamos configurar o GitHub Actions para executar os testes automaticamente a cada push.
