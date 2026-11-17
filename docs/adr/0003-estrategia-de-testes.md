# ADR-0003: Estratégia de testes

- **Status:** Aceito
- **Data:** 2026-11-17
- **Decisores:** Andre Menezes

## Contexto

Código gerado por IA pode parecer correto e falhar em casos de borda. Precisamos de uma rede de segurança rápida, determinística e ligada aos critérios de aceite das SPECs.

## Opções consideradas

1. **Jest + Testing Library + Cypress.**
2. **Vitest + Testing Library + MSW + Playwright + axe.**
3. **Apenas testes E2E.**

## Decisão

Adotamos a **opção 2**:

| Camada         | Ferramenta                | Escopo                                        |
| -------------- | ------------------------- | --------------------------------------------- |
| Unitário       | Vitest                    | Funções puras, schemas, regras de negócio     |
| Componente     | Vitest + Testing Library  | Comportamento e estados de UI                 |
| Integração     | Vitest + MSW              | Componentes + dados (sucesso, erro, latência) |
| Ponta a ponta  | Playwright                | Fluxos críticos de usuário                    |
| Acessibilidade | axe (Vitest e Playwright) | Violações WCAG automatizáveis                 |

Regras:

- Cada `CA-xx` tem ao menos um teste com o identificador no nome.
- Mocks são determinísticos: nada de rede real nem LLM real nos testes.
- A IA pode **propor** casos de borda e testes. O humano valida se o teste verifica o comportamento correto.

## Consequências

### Positivas

- Vitest é rápido e compatível com a configuração do projeto.
- MSW reaproveita os mesmos mocks em testes e desenvolvimento.
- Playwright cobre vários navegadores e tem bom suporte a acessibilidade.

### Negativas e trade-offs

- Mais ferramentas para configurar e manter.
- Algumas partes do App Router (Server Components assíncronos) são mais fáceis de cobrir via E2E do que via teste de componente.

## Como validar

- Pipeline bloqueia merge com teste falhando.
- Toda falha corrigida tem teste de regressão e registro em `docs/testing/failures/`.
