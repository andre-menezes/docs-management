# SPEC-NNN: Nome da funcionalidade

| Campo          | Valor                                           |
| -------------- | ----------------------------------------------- |
| Status         | Rascunho / Em revisão / Aprovada / Implementada |
| Pilar          | 1 a 5 (roadmap)                                 |
| Prioridade     | Alta / Média / Baixa                            |
| ADRs           | ADR-xxxx                                        |
| Depende de     | SPEC-xxx                                        |
| Última revisão | AAAA-MM-DD                                      |

## 1. Contexto e objetivo

Que problema resolve, para quem e qual o resultado esperado.

## 2. Escopo

### Dentro do escopo

-

### Fora do escopo

-

## 3. Histórias de usuário

- **HU-01**: Como `<perfil>`, quero `<ação>` para `<benefício>`.

## 4. Regras de negócio

| ID    | Regra |
| ----- | ----- |
| RN-01 |       |

## 5. Critérios de aceite

```gherkin
Funcionalidade: Nome

  Cenário: CA-01 descrição curta
    Dado que ...
    Quando ...
    Então ...
```

## 6. Restrições

| ID    | Tipo           | Restrição                                          |
| ----- | -------------- | -------------------------------------------------- |
| RT-01 | Técnica        | Ex.: renderização no servidor para a carga inicial |
| RT-02 | Segurança      |                                                    |
| RT-03 | Acessibilidade | Ex.: operável 100% por teclado                     |
| RT-04 | Performance    |                                                    |

## 7. Cenários de erro

| ID     | Situação          | Comportamento esperado | Mensagem ao usuário |
| ------ | ----------------- | ---------------------- | ------------------- |
| ERR-01 | Falha de rede     |                        |                     |
| ERR-02 | Resposta inválida |                        |                     |

## 8. Estados de interface

| Estado     | Descrição / comportamento |
| ---------- | ------------------------- |
| Carregando |                           |
| Vazio      |                           |
| Erro       |                           |
| Sucesso    |                           |

## 9. Contrato de dados

```ts
// Tipos derivados de schemas Zod
```

## 10. Decisão de renderização

Server ou Client Component? Onde os dados são buscados? Qual a estratégia de cache?

## 11. Plano de testes

| Critério | Tipo de teste  | Arquivo |
| -------- | -------------- | ------- |
| CA-01    | Componente     |         |
| ERR-01   | Integração/MSW |         |

## 12. Questões em aberto

- [ ]

## 13. Histórico

| Data       | Alteração |
| ---------- | --------- |
| AAAA-MM-DD | Criação   |
