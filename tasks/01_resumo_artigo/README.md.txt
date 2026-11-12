# 01 — Resumo de artigo científico

## Objetivo

Resumir artigos acadêmicos em quatro seções fixas (objetivo, método,
resultados e conclusão), com controle de tamanho e sem inventar informação.

## Estrutura

- `prompt_v1.md` — versão ingênua (baseline)
- `prompt_v2.md` — versão estruturada
- `prompt_v3.md` — versão few-shot
- `outputs/` — saídas brutas de cada execução

## O que cada versão adiciona

| Versão | O que adiciona | O que se espera |
|---|---|---|
| v1 | Uma linha: "Resuma o artigo abaixo" | Baseline: sem formato, sem limite de tamanho e sem regra de seções |
| v2 | Papel, quatro seções fixas, limite de 180–250 palavras, regra de "Não informado no artigo" e proibição de acrescentar informação | Corrige estrutura e tamanho, mas pode inventar quando falta dado |
| v3 | v2 + três exemplos de entrada e saída, incluindo um artigo com informação faltando | Estabiliza o comportamento no caso difícil |

## Resultados

### v1 — baseline

O modelo devolveu um resumo em texto corrido, sem as quatro seções pedidas.
O conteúdo estava correto: preservou os números do artigo (tamanho da amostra,
escores de fadiga, razão de chances e intervalo de confiança), o instrumento
de medida e a conclusão dos autores.

Falhas observadas:

- **Formato ignorado** — nenhuma seção, apenas parágrafos corridos.
- **Tamanho fora de controle** — o comprimento variou conforme o texto de entrada, e não por instrução.
- **Informação externa acrescentada** — o modelo incluiu uma observação de que o estudo transversal mostra associação, e não causalidade, o que não estava no artigo.

O último ponto é o mais grave: sem instrução explícita, o modelo completou o
texto com conhecimento próprio, o que num resumo científico é uma falha séria.

### v2 — estruturado

As quatro seções apareceram na ordem correta, com os títulos pedidos, e todos
os números foram preservados.

Falhas observadas:

- **Limite de palavras estourado** — o resumo passou de 250 palavras.
- **Seções extensas** — várias seções com 3 a 4 frases, sem margem em relação ao limite.

O v2 resolveu estrutura e formato, mas não o controle de tamanho, que era o
problema escolhido para esta tarefa.

### v3 — few-shot

As quatro seções apareceram corretamente, e o modelo admitiu a ausência de
informação quando necessário:

- Método: "Não informado no artigo o período exato analisado nem o tamanho da amostra."
- Resultados: "Não informado no artigo o número de casos ou a magnitude da diferença."

O modelo não inventou período, amostra nem números, e o tamanho ficou dentro
da faixa pedida.

## Comparação

| Critério | v1 | v2 | v3 |
|---|---|---|---|
| Quatro seções na ordem | não | sim | sim |
| Números preservados | sim | sim | sim |
| Tamanho dentro de 180–250 palavras | não | não | sim |
| Admitiu informação ausente | não | não testado | sim |
| Acrescentou informação externa | sim | não | não |

## Conclusão

A evolução não é linear. O v2 resolveu estrutura e formato, mas falhou no
controle de tamanho, justamente o problema que motivou a tarefa. O v3
resolveu ambos, e o mecanismo foi o exemplo com informação faltando: ao ver
um caso em que a resposta correta era "Não informado no artigo", o modelo
passou a admitir a ausência de dado em vez de preencher a lacuna.

O ganho do few-shot aqui não foi de formato, e sim de **comportamento**: o
exemplo ensinou ao modelo o que fazer quando a informação não está no texto.

## Limitações

- Execução manual, sem automação nem medição estatística.
- Um artigo por versão, e não uma amostra ampla.
- Um único modelo testado.
- Contagem de palavras feita manualmente.

## Como usar

1. Abra o `prompt_vN.md` desejado.
2. Copie o conteúdo para um chat de LLM.
3. Substitua `{texto}` pelo artigo que você quer resumir.
4. Envie e salve a resposta em `outputs/`.

O prompt não depende de um artigo específico: qualquer texto acadêmico
funciona como entrada.