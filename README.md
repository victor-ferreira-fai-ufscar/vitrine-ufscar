# Vitrine UFSCar

Repositório raiz agregador do projeto. O backend e o frontend continuam como repositórios separados e são consumidos aqui como submódulos.

## Submódulos

- Front-end: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-frontend>
  - SSH Keys: <https://git.ufscar.br/-/user_settings/ssh_keys>
- Back-end: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-backend>
  - Documentação (ainda não existe): <https://docs.vitrine.ufscar.br>

## Clone

Para baixar o repositório raiz e os submódulos de uma vez:

```bash
git clone --recurse-submodules https://github.com/victor-ferreira-fai-ufscar/vitrine-ufscar.git
```

Se o clone já existir localmente:

```bash
git submodule update --init --recursive
```

## Como o acesso funciona

O GitHub do root mostra apenas os ponteiros dos submódulos. O conteúdo completo de cada pasta continua vindo do GitLab definido em `.gitmodules`.

- Quem tiver acesso só ao root verá os links dos submódulos, mas não o conteúdo se não tiver permissão nos repositórios do GitLab.
- Quem tiver acesso ao root e aos dois repositórios internos conseguirá clonar tudo com `--recurse-submodules`.

## Deploy

- Deploy (AWS): <https://vitrine-staging.aws.ufscar.br/inicio>
