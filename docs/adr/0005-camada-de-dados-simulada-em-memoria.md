# ADR-0005: Camada de dados simulada em memória

- **Status:** Aceito
- **Data:** 2026-11-17
- **Decisores:** Andre Menezes
- **SPECs relacionadas:** SPEC-000

## Contexto

O projeto não tem backend real e usa apenas dados fictícios. Ainda assim, as telas precisam de carregamento no servidor, mutações, estados de erro e, no pilar 3, de ferramentas (tool calling) que consultem e alterem dados. A solução precisa ser determinística para testes e fácil de substituir por uma API real.

## Opções consideradas

1. **JSON estático importado nos componentes**: simples, mas sem mutações, paginação nem erros.
2. **MSW como "backend" no navegador e no servidor**: ótimo para testes, mas acopla o modelo de dados ao transporte HTTP e fica no lado errado da fronteira para Server Components.
3. **json-server ou SQLite**: processo ou banco extra, além de dependência e configuração adicionais.
4. **Repositório em memória no servidor**, com gerador determinístico e simulação de latência e falhas.

## Decisão

Adotamos a **opção 4**.

- Estado em memória num singleton do servidor, guardado em `globalThis` para sobreviver ao hot reload em desenvolvimento.
- Gerador determinístico com PRNG próprio (sem `@faker-js/faker`), com listas curadas em pt-BR.
- Funções públicas assíncronas que retornam `Result<T, DataError>`, imitando uma API remota.
- O transporte (GraphQL, pilar 2) consome o repositório. **MSW continua sendo usado nos testes de componente** para simular a rede.

## Análise das opções

| Critério                          | JSON estático | MSW | json-server/SQLite | Em memória |
| --------------------------------- | ------------- | --- | ------------------ | ---------- |
| Mutações e regras                 | ❌            | ⚠️  | ✅                 | ✅         |
| Determinismo em testes            | ✅            | ✅  | ⚠️                 | ✅         |
| Dependências extras               | 0             | 1   | 1+                 | 0          |
| Funciona com Server Components    | ✅            | ⚠️  | ✅                 | ✅         |
| Facilidade de trocar por API real | ❌            | ⚠️  | ✅                 | ✅         |

## Consequências

### Positivas

- Zero infraestrutura: `pnpm dev` é suficiente.
- Regras de negócio testáveis isoladamente e reaproveitáveis pelas ferramentas do assistente.
- Interface estável: trocar por uma API real afeta só a implementação do repositório.

### Negativas e trade-offs

- **Os dados voltam ao estado inicial** a cada reinício do servidor.
- **Em ambiente serverless (Vercel)**, cada instância tem a própria memória, então mutações podem "desaparecer" entre requisições. Isso é aceitável para uma demonstração e deve ser comunicado na UI e no README.
- Escrever um gerador próprio dá mais trabalho que usar uma biblioteca.

### Riscos e mitigação

- **Divergência entre a interface simulada e uma API real**: contratos definidos por schemas Zod e documentados na SPEC-000.

## Como validar

- Todos os critérios da SPEC-000 cobertos por testes.
- Nenhuma importação de `src/lib/server/` em Client Components (garantida por `server-only`).
