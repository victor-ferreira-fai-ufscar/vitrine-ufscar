# Frontend — vitrine-frontend

Submódulo: [vitrine-frontend](../vitrine-frontend) ·
Repo: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-frontend>

## Stack

| Item | Versão / Tecnologia |
| ---- | ------------------- |
| Framework | **Angular 18.2** (standalone components + `ApplicationConfig`) |
| Linguagem | TypeScript **5.5** |
| UI | **Bootstrap 5.3**; máscaras com **ngx-mask 18** |
| Reatividade | RxJS 7.8 |
| Lint/Format | ESLint 9 + angular-eslint 18 + **Prettier** 3.3 |
| Testes | Karma + Jasmine |
| Tamanho | ~5.100 linhas (ts/html/scss) · 14 páginas · 37 componentes · 9 serviços |

## Estrutura

```text
src/app/
├── app.component.ts            → shell da aplicação
├── app.config.ts               → providers (router, httpClient, zone coalescing)
├── app.routes.ts               → rotas raiz
├── redirect-external.guard.ts  → guard para redirecionar a URLs externas
├── core/
│   ├── models/      → interfaces (paginação, ODS, departamentos, fale-conosco, ...)
│   └── services/    → 9 serviços HTTP (busca, centros, docentes, ods, temas, ...)
├── pages/           → 14 páginas (uma por seção/público-alvo)
│   ├── buscar/ centros/ docentes/ docente-ufscar/ educador/ empresa/
│   ├── equipe/ fale-conosco/ gestor-publico/ imprensa/ indicadores/
│   └── inicio/ publicacoes/ sobre/
└── shared/
    ├── components/  → componentes reutilizáveis (cabeçalho, rodapé, paginação,
    │                  campo-busca, select, collapse, carregando, ...)
    ├── directives/  → ctrl-c-key.directive
    └── pipes/        → extrair-texto.pipe
```

## Roteamento

- Configurado em [app.routes.ts](../vitrine-frontend/src/app/app.routes.ts) com
  **estratégia de hash** (`useHash: true` → URLs com `#`).
- **Lazy loading** nas seções maiores: `docentes` e `centros` carregam rotas
  filhas via `loadChildren`.
- `RedirectExternalGuard` desvia rotas como `/tecnologias` para uma URL externa
  (`ain.ufscar.br`).
- Restauração de posição de scroll (`withInMemoryScrolling`) e _component input
  binding_ habilitados.
- Rota coringa (`**`) redireciona para `inicio`.

> ℹ️ A estratégia de **hash routing** funciona bem em SPA atrás de qualquer
> servidor estático, mas não é ideal para SEO num portal público cujo objetivo é
> dar **visibilidade/descoberta**. Ver [analise-codigo.md](analise-codigo.md).

## Comunicação com o backend

Os serviços em `core/services` encapsulam as chamadas HTTP e montam a URL a
partir de `environment.backURL`. Exemplo
([busca.service.ts](../vitrine-frontend/src/app/core/services/busca.service.ts)):

```ts
backURL = environment.backURL + '/busca';
listarBusca(busca, offset = 0, max = 10): Observable<IPaginacao<IBusca>> { ... }
listarSugestoes(busca): Observable<IPaginacao<string>> { ... }
```

Os tipos de resposta seguem o contrato `IPaginacao<T>`, espelhando o
`PaginacaoDTO` do backend.

## Ambientes

| Arquivo | `backURL` |
| ------- | --------- |
| `environment.ts` (default) | `http://localhost:8080` |
| `environment.local.ts` | local |
| `environment.dev.ts` | dev |
| `environment.staging.ts` | `https://vitrine-backend-staging.aws.ufscar.br` |
| `environment.prod.ts` | produção |

Scripts em `package.json`: `start` / `start:prod`, `build`, `build:staging`,
`build:dev`, `build:prod`, `lint`, `test`.

## Execução local

```bash
npm install
npm start           # ng serve  (consome http://localhost:8080)
npm run build:staging
```

## Deploy

`Dockerfile` → `chart/` (Helm) + `skaffold.yaml` → GitLab Auto-DevOps
(`BUILD_ENVIRONMENT` por branch) → AWS EKS.

---

_Última revisão: 2026-06-08._
