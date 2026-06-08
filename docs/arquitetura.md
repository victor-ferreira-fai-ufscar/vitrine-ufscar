# Arquitetura — Vitrine UFSCar

## O que é o projeto

A **Vitrine UFSCar** é um portal público que dá visibilidade à produção
acadêmica e à expertise dos **docentes/pesquisadores da UFSCar**. O sistema
permite buscar docentes por nome, tema de atuação, centro/departamento e
Objetivos de Desenvolvimento Sustentável (ODS), além de destacar docentes "em
alta" e conteúdos populares.

O portal é organizado por **públicos-alvo** (gestor público, empresa, educador,
imprensa, docente da própria UFSCar), cada um com sua página de entrada, e
agrega ainda feed do Instagram e formulário de contato ("Fale Conosco").

## Topologia (repositório agregador)

```text
vitrine-ufscar (raiz / agregador — GitHub)
├── docs/                ← contexto técnico (esta pasta)
├── vitrine-backend/     ← submódulo (GitLab)  — Grails / Groovy / Java 17
└── vitrine-frontend/    ← submódulo (GitLab)  — Angular 18 / TypeScript
```

O repositório raiz **não contém código** — apenas ponteiros (commits fixos) para
os dois submódulos hospedados no GitLab interno da UFSCar. Cada submódulo segue a
branch `staging` (desenvolvimento) e `main` (produção).

## Diagrama de componentes

```text
                     ┌─────────────────────────────────────────┐
                     │            Navegador (usuário)           │
                     └───────────────────┬─────────────────────┘
                                         │  HTTPS
                                         ▼
              ┌────────────────────────────────────────────────┐
              │   vitrine-frontend  (Angular 18, SPA)           │
              │   Bootstrap 5 · ngx-mask · roteamento hash      │
              └───────────────────┬────────────────────────────┘
                                  │  REST/JSON  (environment.backURL)
                                  ▼
              ┌────────────────────────────────────────────────┐
              │   vitrine-backend  (Grails 6.2 / Groovy)        │
              │   Controllers → Services → DTOs (JSON views)    │
              └───┬───────────┬───────────┬──────────┬──────────┘
                  │           │           │          │
                  ▼           ▼           ▼          ▼
          ┌───────────┐ ┌───────────┐ ┌────────┐ ┌──────────────┐
          │PostgreSQL │ │Elastic-   │ │ AWS S3 │ │ APIs externas│
          │(GORM/     │ │search 7.8 │ │(fotos, │ │ Google       │
          │Hibernate) │ │(busca)    │ │presign)│ │ Analytics +  │
          │  + views  │ │           │ │        │ │ Instagram    │
          └───────────┘ └───────────┘ └────────┘ └──────────────┘
```

## Fluxo de dados principais

1. **Busca de docentes** — o frontend chama `GET /busca`; o backend consulta o
   **Elasticsearch** (views `docentes_busca_geral_view` / `docentes_dados_gerais_view`
   indexadas com _boosting_ por relevância) e complementa os dados via PostgreSQL
   (GORM Criteria), devolvendo um `PaginacaoDTO`.
2. **Listagem/detalhe de docente** — `GET /docentes` e `GET /docente/{id}`
   consultam o PostgreSQL; as fotos vêm do **S3** como _presigned URLs_ geradas
   sob demanda.
3. **"Em alta"** — o backend consulta a **Google Analytics Data API** para obter
   os IDs de docentes mais acessados e ordena a listagem por essa relevância.
4. **Feed e contato** — `GET /instagram/publicacoes` consome a API do Instagram;
   `POST /fale-conosco/enviar-email` dispara e-mail via Jakarta Mail (SMTP).

## Infraestrutura e deploy

Ambos os submódulos compartilham o mesmo padrão de empacotamento e entrega:

- **Docker** — imagem própria por serviço (backend em `openjdk:17`, usuário
  não-root; frontend servido como build estático).
- **Helm chart** (`chart/`) + **Skaffold** (`skaffold.yaml`) para Kubernetes.
- **GitLab CI/CD** com **Auto-DevOps** e _Container Scanning_, com _build_
  parametrizado por branch (`staging` → ambiente de staging, `main` → produção).
- **Deploy em AWS EKS** (tags de runner `eks`).

| Ambiente | Frontend | Backend |
| -------- | -------- | ------- |
| Staging | (servido via EKS) | `https://vitrine-backend-staging.aws.ufscar.br` |
| Produção | `https://vitrine-staging.aws.ufscar.br/inicio` (deploy atual) | — |

> O `backURL` consumido pelo frontend é definido por ambiente em
> `src/environments/environment.*.ts`.

## Convenções de branch

- `staging` — branch de **desenvolvimento** (foco atual, fixada em `.gitmodules`).
- `main` — branch de **produção**.

---

_Última revisão: 2026-06-08._
