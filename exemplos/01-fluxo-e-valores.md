# Exemplos — Fluxo e valores

**Informativo.** Usam o [contexto comum](00-notacao.md), incluindo saídas, origens e contratos de chamada.

## X-01 — Sobrescrita linear

```air
cell @target : text
^entry:
  a: assign @target <- text("A")
  b: assign @target <- text("B")
  k: invoke program dynamic(read(@target))
```

`RD(@target,before(k))={b}` e `PV={"B"}; remainder=false`. A operação a permanece no inventário e na proveniência, mas não é definição corrente do target. Não há relação especial de “par” entre atribuição e chamada no modelo.

## X-02 — Bifurcação e reconvergência

```air
cell @target : text
cell @flag : bool
^entry:
  a: assign @target <- text("A")
  test: branch read(@flag) then ^yes else ^no
^yes:
  b: assign @target <- text("B")
  jy: jump ^join
^no:
  c: assign @target <- text("C")
  jn: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

A IR conserva duas atribuições alternativas e seus destinos de controle. O CFG deriva a reconvergência em `^join`. `RD={b,c}` e `PV={"B","C"}; remainder=false`. O valor A foi sobrescrito nos dois caminhos. A verdade do flag não precisa ser conhecida para esse resultado de valores possíveis.

## X-03 — Caminho falso sem atualização

```air
cell @target : text
cell @flag : bool
^entry:
  a: assign @target <- text("A")
  test: branch read(@flag) then ^yes else ^join
^yes:
  b: assign @target <- text("B")
  jy: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

`RD={a,b}` e `PV={"A","B"}; remainder=false`. O caminho falso continua diretamente. Não é necessário criar uma operação vazia para representar uma cláusula sintática ausente.

## X-04 — Um ramo termina antes da continuação

```air
cell @target : text
cell @flag : bool
^entry:
  a: assign @target <- text("A")
  test: branch read(@flag) then ^terminate else ^continue
^terminate:
  b: assign @target <- text("B")
  stop: halt normal
^continue:
  c: assign @target <- text("C")
  jc: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

Na entrada de k, `RD={c}` e `PV={"C"}; remainder=false`. Não há caminho de b para k. Uma reconvergência criada apenas porque os ramos pertenciam à mesma construção-fonte violaria a semântica.

## X-05 — Seleção com default explícito

```air
cell @code : int
cell @target : text
^entry:
  d: dispatch read(@code) cases {int(1):^one, int(2):^two} default ^other
^one:
  a: assign @target <- text("ONE")
  j1: jump ^join
^two:
  b: assign @target <- text("TWO")
  j2: jump ^join
^other:
  c: assign @target <- text("OTHER")
  j3: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

`PV={"ONE","TWO","OTHER"}; remainder=false`. Nenhuma alternativa é perdida quando o seletor não corresponde aos casos enumerados. Dois casos semanticamente iguais seriam publicação inválida, não prioridade por ordem de tabela.

## X-06 — Laço de pré-teste e possibilidade de zero iterações

```air
cell @again : bool
cell @target : text
^entry:
  a: assign @target <- text("A")
  start: jump ^test
^test:
  input: havoc.must @again reason EXTERNAL_VALUE_UNKNOWN
  t: branch read(@again) then ^body else ^join
^body:
  b: assign @target <- text("B")
  back: jump ^test
^join:
  k: invoke program dynamic(read(@target))
```

`havoc.must` representa aqui uma entrada booleana cujo valor não foi fornecido, com escrita certa e sem outro efeito. O predicado pode variar por iteração. `RD={a,b}` e `PV={"A","B"}; remainder=false`. A é possível com zero iterações. O desconhecimento de @again não impede o conjunto fechado de @target.

## X-07 — Laço de pós-teste

```air
cell @again : bool
cell @target : text
^entry:
  start: jump ^body
^body:
  b: assign @target <- text("B")
  input: havoc.must @again reason EXTERNAL_VALUE_UNKNOWN
  t: branch read(@again) then ^body else ^join
^join:
  k: invoke program dynamic(read(@target))
```

Todo caminho que chega a k passa por b. `RD={b}` e `PV={"B"}; remainder=false`. A possibilidade de divergência no ciclo não cria um caminho até k sem atribuição.

## X-08 — Cópia captura o valor no ponto da definição

```air
cell @source : text
cell @target : text
^entry:
  a: assign @source <- text("A")
  capture: assign @target <- read(@source)
  b: assign @source <- text("B")
  k: invoke program dynamic(read(@target))
```

`RD(@target,before(k))={capture}`. Para interpretar capture, a análise consulta @source em `before(capture)`, onde sua definição é a. Portanto `PV(@target,before(k))={"A"}`, não B. O operando não é uma expressão preguiçosa reavaliada na chamada.

## X-09 — Bifurcação aninhada com chamada que pode alterar estado

```air
cell @target : text
cell @outer : bool
cell @inner : bool
^entry:
  a: assign @target <- text("A")
  t1: branch read(@outer) then ^yes else ^no
^yes:
  b: assign @target <- text("B")
  t2: branch read(@inner) then ^side else ^join
^side:
  s: invoke program literal("SIDE") effects=may_write(@target) normal ^join
^no:
  c: assign @target <- text("C")
  jn: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

`PV={"B","C"}; remainder=true`. O caminho por s pode produzir outros valores, mas o caminho verdadeiro que não chama s sustenta B e o caminho falso sustenta C. A permanece morta para k. Ignorar os efeitos de s produziria falsa enumeração fechada.

## X-10 — Short-circuit com interação explicitada

```air
cell @left : bool
cell @right : bool
cell @target : text
^entry:
  l: branch read(@left) then ^evaluate_right else ^no
^evaluate_right:
  check: invoke program literal("CHECK") results=[@right]
         effects=none [C-PURE] normal ^right_test
^right_test:
  r: branch read(@right) then ^yes else ^no
^yes:
  a: assign @target <- text("YES")
  jy: jump ^join
^no:
  b: assign @target <- text("NO")
  jn: jump ^join
^join:
  k: invoke program dynamic(read(@target))
```

CHECK só pode executar depois do resultado verdadeiro de l. Seu resultado desconhecido é escrito em @right apenas no retorno normal. Não há chamada escondida em uma expressão booleana pura. Nos caminhos que chegam a k, `PV={"YES","NO"}; remainder=false`.
