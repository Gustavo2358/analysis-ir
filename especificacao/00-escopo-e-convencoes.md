# 00 — Escopo, convenções e garantias

**Analysis IR 1.0.0 — Normativo**

## 1. Vocabulário normativo

**DEVE** e **NÃO DEVE** expressam obrigação e proibição. **DEVERIA** expressa recomendação cuja exceção exige justificativa explícita. **PODE** expressa uma alternativa permitida. A convenção de requisitos é inspirada em BCP 14; estas definições em português governam este conjunto. As [referências](../REFERENCIAS.md) não importam automaticamente requisitos de outras representações.

Os documentos de `especificacao/` e `conformidade/` são normativos. Exemplos e justificativas explicam as regras, mas não as substituem. Em conflito, a regra específica do catálogo prevalece sobre uma descrição geral; um conflito entre regras normativas é defeito da especificação, não liberdade para escolher comportamento.

Identificadores entre crases designam conceitos semânticos. Não prescrevem nomes de classes, métodos ou campos de um formato físico. Cardinalidades como `0..N` são irrestritas pelo contrato; limites operacionais não podem truncar uma publicação silenciosamente.

## 2. Domínio

A V1 representa comportamento imperativo sequencial: leitura e atualização de estado, seleção de caminhos, repetição, transferência, invocação e interação com recursos. Unidades podem possuir múltiplas entradas, estado compartilhado explicitamente e relações de contenção. A contenção lexical não determina transferência de controle nem compartilhamento de memória.

A IR NÃO DEVE exigir uma árvore sintática, tabela de símbolos, resolução nominal, parser, linguagem-fonte ou objeto de análise externo vivo para ser interpretada. Um produtor pode ser um frontend, um tradutor de outra IR ou uma ferramenta que já disponha de fatos semânticos equivalentes.

Concorrência, sinais assíncronos, autoalteração arbitrária de instruções, carregamento irrestrito de código e aritmética de máquina não especificada não possuem semântica precisa no núcleo V1. Quando relevantes, DEVEM aparecer por extensão negociada ou abstração conservadora que inclua seus efeitos. Ausência de perfil não equivale a ausência de comportamento.

## 3. Responsabilidades

A IR DEVE representar operações, operandos, identidades, restrições declarativas de memória, semântica de transferência, contratos de interação, cobertura, incerteza e proveniência. Pode transportar fatos de dependência estrutural já estabelecidos, como uma relação de inclusão de artefato.

A IR NÃO DEVE incorporar como fatos intrínsecos: blocos básicos calculados, arestas computadas de CFG, dominância, alcançabilidade inferida, conjuntos GEN/KILL, conjuntos de reaching definitions, valores propagados, targets calculados de chamadas ou um grafo agregado de dependências. Esses resultados pertencem a publicações derivadas vinculadas à versão exata da IR.

Uma transferência que nomeia destinos na IR é uma regra de comportamento; uma aresta no CFG é a projeção dessa regra. Um contrato externo que limita escritas é uma premissa semântica; o efeito calculado de uma operação é um resultado derivado. A distinção é de autoridade e significado, não de possibilidade de compartilhamento de armazenamento físico.

## 4. Suficiência bilateral

**REQ-BIL-01.** Para cada capacidade admitida, o contrato DEVE identificar: fatos exigidos do produtor, significado preservado, observações disponibilizadas ao consumidor, comportamento quando um fato falta e pelo menos um oráculo observável.

**REQ-BIL-02.** O consumidor NÃO DEVE completar fatos ausentes consultando a linguagem-fonte. O produtor NÃO DEVE antecipar uma análise posterior para aparentar que uma entrada semântica estava resolvida.

**REQ-BIL-03.** A indisponibilidade de uma capacidade no produtor reduz a precisão declarada da publicação; não redefine o significado de uma operação da IR.

## 5. Conservadorismo

Considere `B(P)` o conjunto de comportamentos admitidos pelo programa de entrada, sob premissas explicitadas, e `B(I)` o conjunto de comportamentos representados pela IR. O lowering DEVE satisfazer a inclusão:

```text
abstrair(B(P)) ⊆ B(I)
```

Um lowering exato para um domínio pode satisfazer igualdade. Um lowering conservador pode adicionar possibilidades, mas NÃO DEVE excluir um comportamento possível sem justificativa. Os mecanismos que demonstram essa propriedade não são prescritos; a publicação deve identificar as premissas e os limites de sua alegação.

Um fato de dependência classificado como possível não é prova de execução. Um conjunto finito sem restante desconhecido significa enumeração completa relativamente ao modelo e ao escopo declarados, não prova de que todas as alternativas enumeradas são viáveis no programa concreto.

## 6. Validade, cobertura e precisão são distintas

| Conceito | Pergunta |
| --- | --- |
| Validade da IR | As identidades, tipos, transferências e invariantes são coerentes? |
| Cobertura do produtor | Todo o domínio de entrada alegado foi representado ou explicitamente marcado? |
| Precisão de representação | O comportamento de cada dimensão é exato, limitado conservadoramente ou aberto? |
| Conformidade do consumidor | O consumidor interpreta o perfil e respeita seus limites? |
| Completude de uma consulta | Há algum comportamento relevante que ainda não foi enumerado? |

Uma referência pendente a uma operação inexistente é IR inválida. Uma operação opaca com escopo explícito pode ser IR válida. Essas situações NÃO DEVEM ser confundidas.

## 7. Unidade de execução

O estado abstrato de referência é `(entry, point, memory, localFrames, invocationContext)`. Essa notação define significado; não exige máquina virtual ou estrutura de dados específica.

Cada entrada de unidade cria ou seleciona a instância de armazenamento conforme sua duração declarada. Expressões são avaliadas no estado anterior à operação. A operação aplica sua atualização e sua regra de transferência. Divergência e término anormal são resultados distintos de retorno normal.

## 8. Ausência de garantias implícitas

O núcleo não assume memória independente por nome, inicialização por zero, retorno normal obrigatório, ausência de exceções, pureza de chamada externa, codificação de texto, ordenação alfabética, identidade persistente entre revisões ou completude de catálogo externo. Cada hipótese necessária DEVE ser fato explícito, contrato externo identificado ou incerteza localizada.

## 9. Conformidade

A conformidade é declarada por versão, papel e perfil, conforme [10 — Perfis](10-perfis-de-conformidade.md). O núcleo é fechado semanticamente nesta versão; extensões possuem contratos próprios. Uma declaração genérica “suporta V1” sem papel e perfis é insuficiente.
