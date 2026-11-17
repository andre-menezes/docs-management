# Ciclo de desenvolvimento assistido por IA

O ciclo se baseia no padrão **ReAct** (_Reason + Act_): o agente alterna entre raciocinar, executar uma ação pequena e observar o resultado real (compilador, testes, lint) antes do próximo passo.

```text
┌───────────────┐   ┌─────────────┐   ┌────────────────────────────────┐   ┌─────────────┐
│ 1. Entender   │──▶│ 2. Planejar │──▶│ 3. Implementar (loop ReAct)    │──▶│ 4. Revisar  │
│    a SPEC     │   │             │   │  Raciocinar → Agir → Observar  │   │             │
└───────────────┘   └──────┬──────┘   └───────────────┬────────────────┘   └──────┬──────┘
                    ✋ aprovação humana     🔁 até os testes passarem     ✋ revisão humana
```

## Fase 1: Entender

**Objetivo:** garantir que o agente compreendeu a SPEC antes de qualquer código.

Prompt modelo:

```text
Leia AGENTS.md e docs/specs/SPEC-NNN-*.md. NÃO escreva código ainda.
Responda:
1. Resumo da funcionalidade em até 5 linhas
2. Lista dos critérios de aceite e cenários de erro, com seu entendimento de cada um
3. Ambiguidades, contradições ou lacunas encontradas na SPEC
4. Casos de borda não previstos na SPEC
```

**Saída esperada:** dúvidas respondidas e SPEC ajustada, se necessário.

## Fase 2: Planejar

Prompt modelo:

```text
Com base na SPEC-NNN, proponha um plano de implementação contendo:
- Arquivos a criar/alterar (caminho e responsabilidade)
- Passos ordenados e pequenos, cada um verificável
- Mapeamento CA/ERR → teste
- Decisão Server vs Client Component
- Dependências novas (se houver, com justificativa)
- Riscos
Aguarde minha aprovação antes de implementar.
```

✋ **Checkpoint humano:** o plano respeita os ADRs? Está no escopo? Há dependência nova?

## Fase 3: Implementar (loop ReAct)

Para cada passo do plano:

| Etapa          | O que o agente faz                                                |
| -------------- | ----------------------------------------------------------------- |
| **Raciocinar** | Explica o que vai fazer e por quê, em 1–3 linhas                  |
| **Agir**       | Faz uma alteração pequena (de preferência, teste antes do código) |
| **Observar**   | Roda `pnpm typecheck` e o teste afetado e lê a saída real         |
| **Ajustar**    | Corrige com base na observação, nunca em suposição                |

Regras do loop:

- Um passo por vez. Nada de reescrever vários arquivos às cegas.
- **Limite de 3 tentativas** para o mesmo erro. Depois disso, pare e reporte: hipótese, tentativas e saída do erro.
- Proibido "consertar" um teste alterando a asserção sem justificativa.

## Fase 4: Revisar

1. O agente roda `pnpm validate` e preenche a autorrevisão (`AGENTS.md`, seção 10).
2. O agente entrega o relatório final (`AGENTS.md`, seção 13).
3. ✋ **Revisão humana**, seguindo `security-dependencies-review.md`.
4. Falhas relevantes encontradas no processo são registradas em `docs/testing/failures/`.

## Antipadrões

| Antipadrão                                 | Por que é ruim                            |
| ------------------------------------------ | ----------------------------------------- |
| "Implemente a tela de documentos" sem SPEC | O agente inventa os requisitos            |
| Aceitar código sem rodar os testes         | Código plausível ≠ código correto         |
| Pedir para "fazer os testes passarem"      | Incentiva alterar testes em vez do código |
| Sessões gigantes com muitas features       | Contexto se degrada e o escopo vaza       |
