# Vitrine UFSCar

Repositório **raiz agregador** do projeto Vitrine UFSCar. O backend e o frontend
continuam como repositórios separados no GitLab e são consumidos aqui como
**submódulos**. Este repositório guarda apenas os ponteiros dos submódulos e a
[documentação técnica](docs/) — nunca o código-fonte em si.

## Sobre o projeto

A **Vitrine UFSCar** é um portal público que dá visibilidade à expertise e à
produção dos **docentes/pesquisadores da UFSCar**. Permite buscar docentes por
nome, tema de atuação, centro/departamento e Objetivos de Desenvolvimento
Sustentável (ODS), destacar docentes "em alta" e organizar o conteúdo por
público-alvo (gestor público, empresa, educador, imprensa, docente).

| Componente | Stack | Repositório |
| ---------- | ----- | ----------- |
| Front-end | Angular 18 · TypeScript · Bootstrap 5 | [vitrine-frontend](https://git.ufscar.br/equipe-deotic/vitrine/vitrine-frontend) |
| Back-end | Grails 6.2 · Groovy · Java 17 · PostgreSQL · Elasticsearch | [vitrine-backend](https://git.ufscar.br/equipe-deotic/vitrine/vitrine-backend) |
| Infra | Docker · Helm · GitLab Auto-DevOps · AWS EKS | — |

## Documentação

O contexto técnico completo (arquitetura, API, análise de código) fica em
[`docs/`](docs/), versionado aqui no agregador para poder ser acompanhado **sem
abrir o código-fonte** dos submódulos:

- [docs/arquitetura.md](docs/arquitetura.md) — visão geral, componentes e fluxo de dados
- [docs/backend.md](docs/backend.md) — stack, domínio, API REST e integrações
- [docs/frontend.md](docs/frontend.md) — Angular, páginas, serviços e ambientes
- [docs/analise-codigo.md](docs/analise-codigo.md) — pontos fortes, riscos e recomendações

## Submódulos

Foco na branch `staging` de cada repositório, que é a branch de desenvolvimento.
A branch `main` de cada repositório é a branch de produção.

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

## Se não aparecerem os arquivos da `staging`

Se a pessoa clonar o root e não enxergar os arquivos esperados de
`vitrine-backend` ou `vitrine-frontend`, normalmente é um destes casos:

- Os submódulos não foram inicializados.
- O clone foi feito sem `--recurse-submodules`.
- Falta permissão/autenticação no GitLab dos repositórios internos.

Passos recomendados:

```bash
git submodule sync --recursive
git submodule update --init --recursive
git submodule update --remote --recursive
```

Validação rápida:

```bash
git submodule status
git -C vitrine-backend branch --show-current
git -C vitrine-frontend branch --show-current
```

Se houver erro de autenticação (por exemplo, `HTTP Basic: Access denied`), a
pessoa precisa ter acesso aos dois repositórios no GitLab e configurar credencial
válida (token HTTPS ou chave SSH).

## Como o acesso funciona

O GitHub do root mostra apenas os ponteiros dos submódulos. O conteúdo completo
de cada pasta continua vindo do GitLab definido em `.gitmodules`.

- Quem tiver acesso só ao root verá os links dos submódulos, mas não o conteúdo
  se não tiver permissão nos repositórios do GitLab.
- Quem tiver acesso ao root e aos dois repositórios internos conseguirá clonar
  tudo com `--recurse-submodules`.

## Deploy

- Deploy (AWS): <https://vitrine-staging.aws.ufscar.br/inicio>
