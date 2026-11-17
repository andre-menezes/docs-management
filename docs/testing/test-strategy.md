# Estratégia de testes

Decisão de ferramentas: [ADR-0003](../adr/0003-estrategia-de-testes.md).

## Convenções

- Arquivos: `*.test.ts(x)` ao lado do código. E2E em `e2e/*.spec.ts`.
- Nome do teste referencia a SPEC: `it('CA-02: filtra por status', …)`.
- Estrutura **Arrange / Act / Assert**.
- Queries por acessibilidade: `getByRole` > `getByLabelText` > `getByText` > `getByTestId`.
- Interações com `userEvent`, não com `fireEvent`.
- Mocks de rede via **MSW**, com handlers para sucesso, vazio, erro e latência.

## Uso de IA nos testes

A IA é usada para **ampliar** a cobertura, não para substituir o raciocínio.

Prompt modelo para casos de borda:

```text
Com base na SPEC-NNN, liste casos de borda e de defeito ainda não cobertos
pelos testes em <arquivo>. Para cada caso: entrada, comportamento esperado
segundo a SPEC e por que ele pode falhar. Não escreva código ainda.
```

Categorias a sempre considerar:

- Listas vazias, um item e muitos itens (paginação)
- Strings vazias, muito longas, com acentos, emojis ou HTML
- Datas inválidas, fusos horários, documentos vencidos
- Falha de rede, timeout, resposta malformada, 401/403/500
- Ações repetidas rapidamente (duplo clique) e condições de corrida
- Cancelamento e desmontagem durante carregamento
- Navegação apenas por teclado

Validação humana obrigatória: o teste falha quando o comportamento está errado? Teste de regressão **deve falhar antes da correção**.

## Ciclo de correção de falhas

1. Reproduzir com um teste que falha.
2. Investigar a causa raiz (a IA pode sugerir hipóteses, validadas com evidência).
3. Corrigir.
4. Confirmar que o teste passa e que nada mais quebrou.
5. Registrar em `falhas/FALHA-NNN-descricao.md`.
