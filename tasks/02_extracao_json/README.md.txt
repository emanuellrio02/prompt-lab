# Extração de JSON com LLM

Experimento comparativo de três versões de prompt para extrair três campos estruturados (`nome`, `data`, `valor`) de textos livres e devolver um JSON válido, com tipos e formatos consistentes.

## Estrutura do repositório

```
├── prompt_v1.md   # versão ingênua (baseline)
├── prompt_v2.md   # versão estruturada
├── prompt_v3.md   # versão few-shot
├── input.md       # casos de teste
└── outputs/       # saídas brutas de cada execução
```

## Casos de teste

| ID | Descrição | Texto |
|----|-----------|-------|
| caso_1 | Contrato simples | "Contrato firmado com a Alfa Logística Ltda em 12/03/2025, no valor de R$ 45.000,00." |
| caso_2 | Data incompleta | "Pagamento a João Pereira da Silva, referente a março de 2025, total de 1.250,90 reais." |
| caso_3 | Campo faltando | "Recebemos da Beta Serviços o valor combinado. Data a confirmar." |

## O que cada versão adiciona

| Versão | O que adiciona |
|--------|----------------|
| v1 | Instrução de uma linha, sem schema, sem regras e sem exemplos |
| v2 | Papel, schema com tipos, regra de `null` para campos ausentes e trava de saída |
| v3 | v2 + três exemplos de entrada e saída, incluindo um caso difícil com informação faltando e ambígua |

## Resultados

As saídas marcadas com `—` ainda não foram registradas.

### caso_1: contrato simples

| Versão | Saída |
|--------|-------|
| v1 | `{"nome": "Alfa Logística Ltda", "data": "12/03/2025", "valor": "R$ 45.000,00"}` |
| v2 | `{"nome": "Alfa Logística Ltda", "data": "2025-03-12", "valor": 45000.00, "moeda": "BRL"}` |
| v3 | — |

### caso_2: data incompleta

| Versão | Saída |
|--------|-------|
| v1 | — |
| v2 | `{"nome": "João Pereira da Silva", "data": null, "valor": 1250.90}` |
| v3 | — |

### caso_3: campo faltando

| Versão | Saída |
|--------|-------|
| v1 | — |
| v2 | — |
| v3 | `{"nome": "Beta Serviços", "data": null, "valor": null}` |

## Avaliação

| Critério | v1 | v2 | v3 |
|----------|:--:|:--:|:--:|
| JSON válido | sim | sim | sim |
| Data em ISO 8601 | não | sim | sim |
| Valor como `number` | não | sim | sim |
| Campos ausentes como `null` | não testado | sim | sim |
| Aderência ao schema (sem campos extras) | sim | **não** (`moeda`) | sim |

## Análise da evolução

### v1: ingênuo (baseline)

Instrução de uma linha, sem schema, sem regras, sem exemplos.

Falhas observadas:

- **Data no formato brasileiro** (`12/03/2025`) em vez de ISO 8601.
- **Valor como string**, com símbolo e separador de milhar (`"R$ 45.000,00"`).
- **Nenhuma regra para campos ausentes.**

O v1 serve de baseline: mostra que uma instrução curta gera saída inconsistente e não parseável por um sistema downstream.

### v2: estruturado

Adiciona papel, schema com tipos, regra de `null` e trava de saída.

Ganhos:

- Data convertida para ISO 8601.
- Valor convertido para `number`.
- Campo ausente tratado como `null` (confirmado no caso_2).
- Saída limpa, sem texto explicativo.

Falha observada:

- No caso_1, o modelo acrescentou o campo `moeda: "BRL"`, fora do schema. Mesmo com o schema explícito no prompt, a aderência não foi total.

### v3: few-shot

Adiciona ao v2 três exemplos de entrada e saída, incluindo um caso difícil com informação faltando e ambígua.

Ganhos:

- Caso difícil resolvido corretamente: `null` em data e valor, sem alucinação.
- Schema mantido com exatamente três chaves, sem campos extras.

Custo:

- Prompt maior, com mais tokens por chamada.

## Conclusão

A evolução não é linear em todos os critérios. O v2 já resolve formato e tipos e trata `null` corretamente no caso_2. O ganho real do v3 é **consistência**: os exemplos mostram que a saída tem exatamente três chaves, o que eliminou a tendência do modelo de acrescentar campos, como o `moeda` visto no v2.

Em resumo, o v2 acerta o formato, e o v3 acerta o formato **e** mantém o contrato do schema estável entre casos diferentes.

## Limitações

- Execução manual em interface de chat, sem automação.
- Um único modelo usado nas três versões.
- Amostra pequena: três casos, que não cobrem todos os formatos de entrada possíveis.
- Saídas do v1 e do v3 nos casos 1 e 2 ainda pendentes, o que limita a comparação direta entre versões.

Por isso, os resultados indicam uma tendência e não permitem generalizar.