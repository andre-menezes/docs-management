# Guia de contribuição

## Fluxo de branches

```text
main ──────────────●──────────────────●──────▶  produção (somente via PR de develop)
                   ▲                  ▲
develop ──●────●───┴────●────●────────┴──────▶  integração
           \  /          \  /
            feat/...      fix/...               trabalho diário
```

- `main`: código estável. Recebe merge apenas de `develop` (release) ou `hotfix/*`.
- `develop`: integração. Toda branch de trabalho nasce daqui e volta para cá via Pull Request.

### Nomenclatura

`<tipo>/<id-da-spec>-<descricao-curta-em-kebab-case>` - Descrição em inglês

| Tipo        | Uso                                      | Exemplo                          |
| ----------- | ---------------------------------------- | -------------------------------- |
| `feat/`     | Nova funcionalidade                      | `feat/spec-001-document-list`    |
| `fix/`      | Correção de bug                          | `fix/falha-003-duplicate-paging` |
| `docs/`     | Documentação, ADR ou SPEC                | `docs/adr-0005-cache-strategy`   |
| `refactor/` | Refatoração sem mudança de comportamento | `refactor/extract-hook-filters`  |
| `test/`     | Apenas testes                            | `test/spec-001-edge-cases`       |
| `chore/`    | Configuração, dependências, tooling      | `chore/configure-vitest`         |
| `ci/`       | Pipelines                                | `ci/github-actions-validate`     |
| `hotfix/`   | Correção urgente a partir da `main`      | `hotfix/production-build-error`  |

## Commits: Conventional Commits

```text
<tipo>(<escopo opcional>): <descrição no imperativo, minúscula, sem ponto final>

<corpo opcional: o porquê da mudança>

<rodapé opcional: Refs: SPEC-001 | BREAKING CHANGE: ...>
```

Tipos: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.

Exemplos:

```text
feat(documents): adiciona paginação na listagem

docs(adr): registra decisão sobre estratégia de cache

fix(assistant): interrompe streaming ao desmontar componente

Refs: SPEC-005, FALHA-002
```

Commits gerados com ajuda de IA seguem o mesmo padrão. O humano é responsável por revisar e assinar.

## Pull Requests

- Destino: `develop` (exceto release e hotfix).
- Título no formato de commit: `feat(documents): listagem com paginação`.
- A descrição deve conter: SPEC relacionada, resumo, critérios de aceite atendidos, como testar, evidências (prints ou saída dos testes) e riscos.
- Um PR = uma SPEC ou um objetivo. PRs pequenos.

## Definition of Done

- [ ] SPEC com status `Aprovada` e todos os critérios de aceite atendidos
- [ ] Testes cobrindo cada critério de aceite e cenário de erro
- [ ] `pnpm validate` passando
- [ ] Revisão humana do código gerado por IA (comportamento, dependências, segurança)
- [ ] Acessibilidade verificada
- [ ] ADR criado ou atualizado se houve decisão arquitetural
- [ ] Falhas relevantes registradas em `docs/testing/failures/`
