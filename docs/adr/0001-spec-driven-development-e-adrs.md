# ADR-0001: Spec Driven Development e ADRs

- **Status:** Aceito
- **Data:** 2026-11-17
- **Decisores:** Andre Menezes

## Contexto

O projeto será desenvolvido com forte apoio de agentes de IA. Sem especificações explícitas, agentes tendem a inventar requisitos, ampliar o escopo e produzir código difícil de validar. Também precisamos explicar, em entrevistas e no portfólio, **por que** cada decisão técnica foi tomada.

## Opções consideradas

1. **Prompts livres**: descrever a tarefa direto no chat do agente.
2. **Issues do GitHub** como única fonte de requisitos.
3. **Spec Driven Development (SDD)**: SPECs versionadas no repositório, com critérios de aceite, restrições e erros, mais ADRs para decisões arquiteturais.

## Decisão

Adotamos a **opção 3**.

- Toda funcionalidade nasce de uma SPEC em `docs/specs/` com status `Aprovada` antes da implementação.
- Critérios de aceite são numerados (`CA-xx`), escritos em Gherkin e rastreáveis até os testes.
- Cenários de erro (`ERR-xx`) e restrições (`RT-xx`) são explícitos.
- Decisões arquiteturais significativas são registradas como ADR em `docs/adr/`. ADRs são imutáveis: para mudar uma decisão, cria-se um novo ADR que substitui o anterior.

## Consequências

### Positivas

- O agente tem contexto preciso e verificável. A revisão humana compara o resultado com a SPEC, não com a memória.
- Rastreabilidade: SPEC → critério → teste → commit.
- Documentação serve como material de portfólio e entrevista.

### Negativas e trade-offs

- Custo inicial de escrita antes de codar.
- Risco de a SPEC ficar desatualizada.

### Riscos e mitigação

- **SPEC desatualizada**: mudanças de comportamento exigem atualizar a SPEC no mesmo PR (item da Definition of Done).

## Como validar

- 100% dos PRs de funcionalidade referenciam uma SPEC.
- Todo `CA-xx` aparece em ao menos um nome de teste.
