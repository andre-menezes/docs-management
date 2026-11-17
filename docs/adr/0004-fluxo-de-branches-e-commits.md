# ADR-0004: Fluxo de branches e commits

- **Status:** Aceito
- **Data:** 2026-11-17
- **Decisores:** Andre Menezes

## Contexto

Precisamos de um histórico legível que mostre a evolução do projeto e rastreie cada mudança até uma SPEC, inclusive mudanças feitas por agentes de IA.

## Opções consideradas

1. **Trunk-based**: tudo em `main` com branches curtas.
2. **Git Flow completo**: `main`, `develop`, `release/*`, `hotfix/*`, `feature/*`.
3. **Git Flow simplificado**: `main` + `develop` + branches tipadas, com Conventional Commits.

## Decisão

Adotamos a **opção 3**, detalhada em `CONTRIBUTING.md`:

- `main` (estável) e `develop` (integração) são protegidas: sem push direto.
- Branches de trabalho nascem de `develop` com o padrão `<tipo>/<id>-<descricao>`.
- Commits seguem Conventional Commits, com `Refs: SPEC-xxx` no rodapé quando aplicável.

## Consequências

### Positivas

- Histórico semântico, o que permite gerar changelog automaticamente no futuro.
- `develop` funciona como ambiente de preview antes da produção.

### Negativas e trade-offs

- Mais cerimônia que trunk-based para um projeto individual. Aceitamos esse custo porque simula o fluxo de equipes reais.

## Como validar

- Lint de commits (commitlint) no CI, previsto para o pilar 5.
