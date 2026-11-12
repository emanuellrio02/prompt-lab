# Laboratório de Prompts e Experimentos com LLM

Repositório de experimentação controlada com prompts de LLM.

O objetivo é testar como a qualidade de uma instrução afeta a qualidade da
saída de um modelo de linguagem, usando a mesma tarefa, o mesmo modelo e o
mesmo tipo de texto de entrada, variando apenas o prompt.

## Metodologia

Cada tarefa é resolvida em três versões de prompt, do mais simples ao mais
elaborado. A comparação entre elas mostra o que cada elemento adicionado
resolveu e o que não resolveu.

| Versão | O que adiciona | O que se espera |
|---|---|---|
| v1 | Instrução curta, sem regras nem exemplos | Baseline: costuma falhar em formato, tipo e campos ausentes |
| v2 | Papel, schema com tipos, regras explícitas e trava de saída | Corrige formato e vocabulário, mas pode errar a calibragem |
| v3 | v2 + exemplos de entrada e saída (few-shot) | Estabiliza o comportamento nos casos difíceis |

## Tarefas

| # | Tarefa | Status |
|---|---|---|
| 01 | Resumo de artigo científico | concluída |
| 02 | Extração de JSON | concluída |
| 03 | Classificação de tickets de SAC | concluída |

## Estrutura

```
.
├── tasks/
│   ├── 01_resumo_artigo/
│   │   ├── input.md
│   │   ├── prompt_v1.md
│   │   ├── prompt_v2.md
│   │   ├── prompt_v3.md
│   │   ├── outputs/
│   │   └── README.md
│   ├── 02_extracao_json/
│   │   └── ...
│   └── 03_classificacao_tickets/
│       └── ...
├── .gitignore
└── README.md
```

Cada pasta de tarefa contém os três prompts, as saídas brutas do modelo e um
README próprio com a análise daquela tarefa.

## Resultados

### 01 — Resumo de artigo científico

Tarefa: resumir artigos acadêmicos em quatro seções fixas, com controle de
tamanho e sem inventar informação.

O v1 devolveu texto corrido, sem as seções pedidas, e acrescentou uma
observação que não estava no artigo, o que é uma falha grave num resumo
científico. O v2 acertou estrutura e formato, mas estourou o limite de
palavras. O v3 acertou os dois, e o mecanismo foi o exemplo com informação
faltando: ao ver um caso em que a resposta correta era "Não informado no
artigo", o modelo passou a admitir a ausência de dado em vez de preencher a
lacuna.

Conclusão: o ganho do few-shot aqui não foi de formato, e sim de
comportamento.

### 02 — Extração de JSON

Tarefa: extrair três campos estruturados de textos livres, com tipos e
formatos consistentes.

O v1 devolveu data no formato brasileiro e valor como texto. O v2 corrigiu
formato e tipos, mas inventou um campo fora do schema. O v3 manteve o schema
estável nos três casos, inclusive no caso com informação faltando.

Conclusão: o v2 acerta o formato, e o v3 acerta o formato e mantém o contrato
do schema estável entre casos diferentes.

### 03 — Classificação de tickets de SAC

Tarefa: classificar tickets em categoria, prioridade e sentimento, com
prioridade calibrada pelo impacto real e não pelo tom do cliente.

O v1 devolveu texto livre em seis campos e confundiu risco comercial com
impacto operacional. O v2 acertou o caso simples, mas subestimou um problema
crítico porque o cliente escreveu com tom calmo. O v3 corrigiu esse caso: o
exemplo ensinou que impacto grave pode vir acompanhado de tom controlado.

Conclusão: o v2 resolve a calibragem em casos simples, mas o few-shot é
necessário quando o impacto é grave e o tom é calmo.

## O que os três experimentos mostram juntos

O padrão se repete nas três tarefas: o v2 resolve formato e vocabulário, mas
não resolve comportamento. Ele acerta a estrutura da saída e falha nos casos
em que o modelo precisa decidir algo que a regra escrita não cobre, como
inventar informação ausente, manter o schema fechado ou calibrar prioridade
pelo impacto.

O few-shot resolve isso porque o exemplo não descreve a regra, ele a
demonstra. É a diferença entre dizer "não invente" e mostrar uma saída em que
a resposta correta é "não informado".

## Como reproduzir

1. Abra o `prompt_vN.md` da tarefa.
2. Copie o conteúdo para um chat de LLM.
3. Substitua `{texto}` pelo texto que você quer processar.
4. Salve a resposta bruta em `outputs/`.
5. Registre a avaliação no README da tarefa.

A execução é manual, sem uso de API paga. O modelo foi mantido constante
entre as três versões, para que a diferença medida seja do prompt e não do
modelo.

## Limitações

- Execução manual, sem automação nem medição estatística.
- Amostra pequena: dois a três casos por tarefa.
- Um único modelo testado.
- Três tarefas desenvolvidas, e não um conjunto amplo.
- Em alguns casos, as versões foram executadas com textos diferentes, o que
  enfraquece a comparação direta.
