# 03 — Classificação de tickets de SAC

## Objetivo

Classificar tickets de atendimento ao cliente em três campos (categoria,
prioridade e sentimento), devolvendo JSON válido, com prioridade calibrada
pelo impacto real do problema e não pelo tom do cliente.

## Estrutura

- `prompt_v1.md` — versão ingênua (baseline)
- `prompt_v2.md` — versão estruturada
- `prompt_v3.md` — versão few-shot
- `outputs/` — saídas brutas de cada execução

## O que cada versão adiciona

| Versão | O que adiciona | O que se espera |
|---|---|---|
| v1 | Uma linha: "Classifique o ticket abaixo" | Baseline: sem categorias definidas, sem escala de prioridade e sem schema |
| v2 | Papel, JSON de três campos, listas fechadas de categoria, prioridade e sentimento, e regra de impacto vs. tom | Corrige formato e vocabulário, mas pode errar a calibragem de prioridade |
| v3 | v2 + cinco exemplos, incluindo cliente irritado com dúvida simples e impacto grave com tom calmo | Estabiliza a calibragem de prioridade |

## Resultados

### Caso 1 — dúvida com tom irritado

Ticket de pré-venda sobre compatibilidade de produto, com cliente na terceira
tentativa de contato e ameaça de comprar da concorrência.

**v1** devolveu texto livre em seis campos (tipo, prioridade, sentimento,
recorrência, risco comercial e ação sugerida). Classificou a prioridade como
`alta`, justificando: "prazo até amanhã e risco de perder a venda".

**v2** devolveu JSON válido com `prioridade: baixa`.

**v3** devolveu JSON válido com `prioridade: baixa`.

O v1 errou ao ler risco comercial como impacto operacional. A ameaça de
perder a venda pressiona o negócio, não o cliente, e a regra de prioridade
considera o impacto do problema sobre quem abriu o ticket.

Aqui o v2 acertou sozinho: a regra "baseie a prioridade no impacto, não no
tom" foi suficiente, e o few-shot não foi necessário.

### Caso 2 — problema técnico crítico

Ticket de falha de sincronização do sistema de ponto, com 340 funcionários
afetados e fechamento de folha no dia seguinte.

**v1** devolveu texto livre em seis campos. Classificou a prioridade como
`crítica`.

**v2** devolveu JSON válido com `prioridade: alta`.

**v3** devolveu JSON válido com `prioridade: critica`.

Este é o caso que justifica o few-shot. O ticket descreve serviço totalmente
indisponível com prazo crítico, que é a definição de `critica` no próprio
prompt. O v2 subestimou porque o cliente escreveu com tom controlado, sem
reclamação. O v3 reconheceu o padrão porque o Exemplo 2 do prompt descreve
exatamente esse cenário.

## Comparação

| Critério | v1 | v2 | v3 |
|---|---|---|---|
| JSON válido | não | sim | sim |
| Campos do schema (3) | não (6 campos) | sim | sim |
| Caso 1 — prioridade correta | não (`alta`) | sim (`baixa`) | sim (`baixa`) |
| Caso 2 — prioridade correta | sim (`crítica`) | não (`alta`) | sim (`critica`) |
| Sentimento no caso 2 | fora da lista | `negativo` | `muito_negativo` |

## Conclusão

A hipótese inicial era que o v1 marcaria tudo como prioridade alta e que o v3
resolveria. Os dados mostraram algo mais específico.

O v1 não marcou tudo como alta: acertou o caso 2. O erro dele foi outro,
confundir risco comercial com impacto operacional no caso 1. E, como escreveu
texto livre, não precisou escolher dentro de uma escala fechada, o que
facilitou o acerto.

O v2 resolveu formato e vocabulário e acertou a calibragem no caso simples,
mas subestimou o caso grave: quando o cliente escreve com tom calmo sobre um
problema crítico, o modelo lê o tom e não o impacto.

O v3 corrigiu isso. O ganho do few-shot não foi de formato, e sim de
**calibragem**: o exemplo ensinou que impacto grave pode vir acompanhado de
tom controlado.

## Limitações

- Execução manual, sem automação nem medição estatística.
- Dois casos testados, e não uma amostra ampla.
- Um único modelo testado.
- O caso 5 (ticket ambíguo entre duas categorias) não foi executado.
- Nas saídas registradas, as aspas aparecem como curvas (`“` `”`) em vez de
  retas, o que torna o JSON inválido para parse automático.

## Como usar

1. Abra o `prompt_vN.md` desejado.
2. Copie o conteúdo para um chat de LLM.
3. Substitua `{texto}` pelo ticket que você quer classificar.
4. Envie e salve a resposta em `outputs/`.

O prompt não depende de um ticket específico: qualquer texto de SAC funciona
como entrada. As categorias e prioridades podem ser ajustadas no próprio
prompt para outro domínio.