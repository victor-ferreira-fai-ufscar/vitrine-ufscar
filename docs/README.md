# Documentação — Vitrine UFSCar

Esta pasta concentra o **contexto técnico** dos dois submódulos do projeto
([vitrine-backend](../vitrine-backend) e [vitrine-frontend](../vitrine-frontend)),
para que seja possível acompanhar a arquitetura e a evolução do código **sem
precisar abrir o código-fonte** de cada repositório do GitLab.

> A documentação vive no repositório raiz (agregador) porque ele só guarda os
> ponteiros dos submódulos — nunca o código-fonte em si. Assim, o contexto fica
> versionado e acessível sem expor os repositórios internos.

## Índice

| Documento | Conteúdo |
| --------- | -------- |
| [arquitetura.md](arquitetura.md) | Visão geral do sistema, como front, back e infra se conectam, e o fluxo de dados |
| [backend.md](backend.md) | Stack, domínio, API REST, integrações (Elasticsearch, S3, Google Analytics, Instagram) |
| [frontend.md](frontend.md) | Stack Angular, páginas, serviços, roteamento e ambientes |
| [analise-codigo.md](analise-codigo.md) | Pontos fortes, riscos e recomendações encontradas na leitura do código |

## Como manter atualizado

Os documentos refletem o estado do código na data indicada no rodapé de cada um.
Sempre que os submódulos forem atualizados para uma nova ponta da `staging`:

```bash
git submodule update --remote --recursive
```

Vale revisar se a arquitetura ou a superfície de API mudaram e ajustar os
documentos correspondentes.

---

_Última revisão: 2026-06-08 — gerada a partir da leitura do código nas pontas
`staging` de ambos os submódulos._
