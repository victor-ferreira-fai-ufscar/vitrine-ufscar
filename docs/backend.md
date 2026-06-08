# Backend — vitrine-backend

Submódulo: [vitrine-backend](../vitrine-backend) ·
Repo: <https://git.ufscar.br/equipe-deotic/vitrine/vitrine-backend>

## Stack

| Item | Versão / Tecnologia |
| ---- | ------------------- |
| Framework | Grails **6.2.0** (perfil `rest_api`) |
| Linguagem | Groovy + Java **17** |
| Build | Gradle (wrapper) + plugin Grails 6.1.2 |
| ORM | GORM / **Hibernate 5** (`ddl auto: none` — schema gerenciado fora da app) |
| Banco | **PostgreSQL** (driver 42.7.7); H2 apenas em runtime de teste |
| Busca | **Elasticsearch 7.8.0** via plugin `grails elasticsearch 3.0.1` |
| Armazenamento | **AWS S3** (`aws-java-sdk-s3`) — fotos via _presigned URLs_ |
| Métricas | **Google Analytics Data API** (`google-analytics-data`) |
| Integrações | API do **Instagram**; e-mail via **Jakarta Mail** + Angus Mail |
| Views JSON | plugin `views-json` (renderização da API) |
| Testes | Spock + `grails-gorm-testing-support` |
| Tamanho | ~3.100 linhas de Groovy |

## Estrutura

```text
grails-app/
├── controllers/vitrine/backend/   → camada HTTP (9 controllers + UrlMappings)
├── services/vitrine/backend/      → regra de negócio (11 services)
├── domain/vitrine/backend/        → domínio GORM (10 classes, várias = views de BD)
└── conf/                          → application.yml, logback.xml
src/main/groovy/vitrine/backend/
├── commands/    → validação de entrada (BuscaCommand, DocenteListaCommand, ...)
├── dtos/        → objetos de resposta (PaginacaoDTO, DocenteListaDTO, ...)
├── constants/   → TemasConstants, TempoConstants
├── enums/       → OrderEnum, TipoUnidadeOrganizacionalEnum
└── utils/       → StringUtils
src/main/docker/ → docker-compose, elasticsearch-compose, init.sql, entrypoint
```

A separação **Controller → Service → DTO**, com `Command` objects validando a
entrada, é consistente em todo o backend.

## Domínio

Boa parte do domínio mapeia **views do PostgreSQL** (somente leitura,
`version false`), não tabelas — o schema é mantido fora da aplicação:

- `DocentesBuscaGeralView` / `DocentesDadosGeraisView` — projeções para listagem
  e busca de docentes (as principais; a primeira é indexada no Elasticsearch com
  _boosting_: `nomePessoa` 3.0, `pessoaTemaTitulo` 2.0, `pessoaTemaConteudo` 1.5).
- `Tema` / `PessoaTema` — temas de atuação e o vínculo docente↔tema.
- `ObjetivoDesenvolvimentoSustentavel` / `PessoaObjetivoDesenvolvimentoSustentavel`
  — ODS e o vínculo docente↔ODS.
- `UnidadeOrganizacional` / `TipoUnidadeOrganizacional` / `Campus` — estrutura
  organizacional (centros, departamentos).
- `TopicosPopulares` — tópicos em destaque.

## Superfície de API (UrlMappings)

| Método | Rota | Controller/ação | Função |
| ------ | ---- | --------------- | ------ |
| GET | `/busca` | busca/busca | Busca geral de docentes (Elasticsearch) |
| GET | `/busca/sugestao` | busca/sugestao | Autocomplete de termos |
| GET | `/centros` | centro/lista | Lista de centros |
| GET | `/centro/{id}/departamentos` | centro/departamentos | Departamentos de um centro |
| GET | `/docentes` | docente/lista | Listagem paginada/filtrada de docentes |
| GET | `/docente/{id}` | docente/detalhes | Detalhe de um docente |
| POST | `/fale-conosco/enviar-email` | faleConosco/enviarEmail | Envio de e-mail de contato |
| GET | `/instagram/publicacoes` | instagram/publicacoes | Feed do Instagram |
| GET | `/ods` | ods/lista | Lista de ODS |
| GET | `/tema/home` | tema/home | Temas da home |
| GET | `/tema/lista/centro` | tema/listaPorCentro | Temas por centro |
| GET | `/topicos-populares` | topicoPopular/lista | Tópicos populares |
| POST | `/elastc-search/reindex` | elastcSearchReindex/reindex | Reindexação do Elasticsearch |

> ⚠️ Os recursos REST genéricos (`GET/POST/PUT/DELETE /$controller/...`) também
> ficam ativos pelas regras padrão do início do `UrlMappings`. Vale conferir se
> isso expõe ações não intencionais. Ver [analise-codigo.md](analise-codigo.md).

## Configuração (application.yml)

- **Segredos externalizados** via variáveis de ambiente (`${...}`): credenciais
  de banco, AWS, Google Analytics, Instagram e SMTP. Nenhum segredo versionado. 👍
- **CORS** habilitado, com origem do frontend parametrizada (`${URL_FRONTEND}`),
  `allowCredentials: true`.
- Perfis de ambiente: `local`, `development`, `homolog`, `production` (este com
  pool de conexões ajustado: `validationQuery`, `testOnBorrow`, `removeAbandoned`).
- Elasticsearch configurado em modo `transport`/`http`, cluster
  `vitrine-elasticsearch`, sem reindex automático no startup.

## Build e execução local

```bash
# requer Java 17, PostgreSQL e Elasticsearch (ver src/main/docker)
./gradlew bootRun
```

O `bootRun` já injeta os `--add-opens` necessários para o Groovy/Grails no Java
17 e usa o profile passado em `spring.profiles.active`.

## Deploy

`Dockerfile` (openjdk 17, usuário não-root, entrypoint próprio) → `chart/` (Helm)
→ GitLab Auto-DevOps → AWS EKS. Build parametrizado por branch
(`GRAILS_ENV=production` no job de build).

---

_Última revisão: 2026-06-08._
