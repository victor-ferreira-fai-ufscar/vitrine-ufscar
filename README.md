# Vitrine UFSCar

Repositório raiz agregador do projeto. O backend e o frontend continuam como repositórios separados e são consumidos aqui como submódulos.

## Submódulos

Foco na branch `staging` de cada repositório, que é a branch de desenvolvimento. A branch `main` de cada repositório é a branch de produção.

- Front-end: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-frontend>
  - Staging: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-frontend/-/tree/staging>
  - SSH Keys: <https://git.ufscar.br/-/user_settings/ssh_keys>
- Back-end: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-backend>
  - Staging: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-backend/-/tree/staging>
  - Documentação (ainda não existe): <https://docs.vitrine.ufscar.br>

## Clone

Para baixar o repositório raiz e os submódulos de uma vez:

```bash
git clone --branch staging --recurse-submodules https://github.com/victor-ferreira-fai-ufscar/vitrine-ufscar.git
```

Se o clone já existir localmente:

```bash
git submodule update --init --recursive
```

Para atualizar os submódulos para a ponta da `staging` configurada em `.gitmodules`:

```bash
git submodule update --remote --recursive
```

## Se nao aparecer os arquivos da `staging`

Se a pessoa clonar o root e nao enxergar os arquivos esperados de `vitrine-backend` ou `vitrine-frontend`, normalmente e um destes casos:

- Os submodulos nao foram inicializados.
- O clone foi feito sem `--recurse-submodules`.
- Falta permissao/autenticacao no GitLab dos repositorios internos.

Passos recomendados:

```bash
git submodule sync --recursive
git submodule update --init --recursive
git submodule update --remote --recursive
```

Validacao rapida:

```bash
git submodule status
git -C vitrine-backend branch --show-current
git -C vitrine-frontend branch --show-current
```

Se houver erro de autenticacao (por exemplo, `HTTP Basic: Access denied`), a pessoa precisa ter acesso aos dois repositorios no GitLab e configurar credencial valida (token HTTPS ou chave SSH).

## Como o acesso funciona

O GitHub do root mostra apenas os ponteiros dos submódulos. O conteúdo completo de cada pasta continua vindo do GitLab definido em `.gitmodules`.

- Quem tiver acesso só ao root verá os links dos submódulos, mas não o conteúdo se não tiver permissão nos repositórios do GitLab.
- Quem tiver acesso ao root e aos dois repositórios internos conseguirá clonar tudo com `--recurse-submodules`.

## Deploy

- Deploy (AWS): <https://vitrine-staging.aws.ufscar.br/inicio>
