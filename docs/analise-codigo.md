# Análise de código — Vitrine UFSCar

Leitura realizada nas pontas `staging` de ambos os submódulos em 2026-06-08.
As observações estão classificadas por severidade. Caminhos apontam para dentro
dos submódulos.

## Pontos fortes

- **Arquitetura em camadas clara** no backend: `Controller → Service → DTO`, com
  `Command` objects validando a entrada antes de chegar ao serviço.
- **Segredos externalizados** via variáveis de ambiente no `application.yml` —
  nada sensível versionado.
- **Busca bem modelada**: uso de Elasticsearch com _boosting_ por campo e views
  de banco dedicadas para projeção, separando leitura de busca da escrita.
- **Segurança de container**: o backend roda como usuário não-root no Docker.
- **Frontend organizado** por `core` / `pages` / `shared`, com lazy loading nas
  seções maiores e lint/format (ESLint + Prettier) configurados.
- **CI/CD padronizado** entre os dois repositórios (Auto-DevOps + container
  scanning), com build parametrizado por branch.

## 🔴 Riscos / possíveis bugs

### 1. Paginação com `Collections.shuffle` é não-determinística
`vitrine-backend` · `DocenteService.executarBuscaCriteria` (ramo `emAlta == false`)

```groovy
List<Long> dbIdsUnicos = dbIds.unique()
Collections.shuffle(dbIdsUnicos)        // ← embaralha a cada requisição
...
List<Long> pagedIds = dbIdsUnicos.subList(cmd.offset, limit)
```

O embaralhamento acontece **a cada chamada**. Como a paginação fatia a lista por
`offset`, duas páginas consecutivas são embaralhadas de forma independente — o
usuário pode ver **docentes repetidos entre páginas e outros nunca aparecerem**.
Se a intenção é ordem aleatória, ela precisa ser **estável dentro de uma sessão
de paginação** (ex.: _seed_ fixo derivado da query, ou ordenação determinística
no banco com `ORDER BY ... , id`).

### 2. Rotas REST genéricas abertas por padrão
`vitrine-backend` · `UrlMappings.groovy:5-10`

As regras padrão (`delete/get/post/put/patch "/$controller/..."`) ficam ativas
para **todos** os controllers, além das rotas nomeadas. Vale verificar se isso
expõe ações de escrita/leitura não intencionais (ex.: `DELETE /docente/{id}`).
Recomenda-se restringir explicitamente ao que a API deve oferecer.

### 3. `POST /elastc-search/reindex` sem proteção aparente
`vitrine-backend` · `UrlMappings.groovy:33`

A reindexação do Elasticsearch é uma operação cara e administrativa. Não há
indício de autenticação/autorização nessa rota — convém protegê-la (rede
interna, token de admin, ou removê-la do roteamento público).

## 🟡 Manutenibilidade

### 4. `DocenteService.lista` / `executarBuscaCriteria` muito complexos
`vitrine-backend` · `DocenteService.groovy`

A lógica de "em alta" + "explorando fronteiras" tem três blocos de
`createCriteria` praticamente **duplicados** (projeções, alias `pt`, filtros de
centro/ODS repetidos em `executarBuscaCriteria`, `completarExplorandoFronteiras`
e `aplicarFiltrosDinamicos`). É o trecho de maior risco de regressão. Sugestões:
extrair a montagem de projeção/critério para um método único reutilizável e
cobrir o comportamento de paginação com testes.

### 5. Typo propagado para a API: `elastc-search` / `ElastcSearch`
`vitrine-backend` · controller `ElastcSearchReindexController` e rota
`/elastc-search/reindex`

Falta o "i" de "Elastic". Como já está na superfície de API, renomear exige
atenção (é _breaking change_ para quem consome). Documentado aqui para não se
perder.

### 6. Mistura de métodos `static` e de instância em `BuscaService`
`vitrine-backend` · `BuscaService.groovy`

`busca()` é de instância, mas `sugestao()`, `buscaGeralDocentes()` etc. são
`static`. Em um _service_ Grails (singleton, injetável), o uso de `static`
dificulta testes e mocking — padronizar como instância é preferível.

### 7. Hash routing num portal de descoberta
`vitrine-frontend` · `app.routes.ts:87` (`useHash: true`)

URLs com `#` não são bem indexadas por buscadores. Para um portal cujo objetivo é
**visibilidade**, considerar `PathLocationStrategy` (URLs limpas) + configuração
de _fallback_ no servidor, e eventualmente SSR/prerender para SEO.

## 🟢 Pequenos ajustes / limpeza

- **Arquivos órfãos** nos submódulos: `README (cópia).md` (back e front) e
  `vitrine-frontend/Dockerfile.OLD` parecem sobras e podem ser removidos.
- **READMEs dos submódulos** ainda são o **template padrão do GitLab** (texto
  "To make it easy for you to get started…"), sem conteúdo real do projeto.
- **Typo** em `environment.*.ts` do frontend: `analitycsID` (deveria ser
  `analyticsID`) — e o valor está vazio em staging.
- **Flag `-noverify`** no `bootRun` (`build.gradle:96`) está depreciada no Java
  17 e tende a só gerar warning; pode ser removida.
- **N _presigned URLs_ por página** em `DocenteService.lista` (uma por docente).
  A geração é local (sem rede), então o impacto é pequeno, mas vale ter em mente
  se a página crescer.

## Sugestão de priorização

1. Corrigir a paginação aleatória (item 1) — afeta diretamente o usuário final.
2. Revisar exposição das rotas REST genéricas e proteger o `reindex` (itens 2 e 3).
3. Refatorar/cobrir com testes o `DocenteService` (item 4).
4. Os demais itens são incrementais e podem entrar em limpezas de rotina.

---

_Esta análise é um retrato do código na data acima; reavalie após
`git submodule update --remote`._
