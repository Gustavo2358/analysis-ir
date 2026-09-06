# Exemplos — Conhecimento de tipo

**Informativo.** Usam o [contexto comum](00-notacao.md). As regras são definidas em [02 — Tipos](../especificacao/02-tipos-valores-e-operandos.md) e [04 — Operações](../especificacao/04-operacoes.md); os exemplos não criam operações novas. `u`, `v` e `r` abaixo são identidades de lacunas declaradas, com origem derivada do cenário, motivo e escopo indicados. Cada ocorrência de operando tem identidade própria e origem do uso, separadas da declaração.

## X-31 — Três estados de conhecimento de domínio

```air
uncertainty u: TYPE_UNKNOWN scope={@untyped} origin=example:X-31/type
cell @name : known(text)
cell @token : known(opaque_type(example.token,1))
cell @untyped : unknown_type(u)
^entry:
  a: assign @name <- text("A")
  h: havoc.must @token reason EXTERNAL_VALUE_UNKNOWN
  end: jump ^done
```

O contrato ilustrativo `example.token@1` identifica um domínio de tokens sem igualdade, aritmética, representação física ou conversão para texto implicitamente disponíveis. A publicação declara essa capacidade e seu modo de consumo; para este cenário, leitura, cópia de valor em células e `havoc` conservam o domínio sem interpretar tokens. Nenhuma operação precisa de extensão é executada.

`@name` tem domínio `text`; `@token` tem domínio identificado de extensão, mesmo com conteúdo desconhecido; `@untyped` tem célula e objeto identificados, mas domínio não estabelecido. Seu estado de entrada é `uninitialized`, conforme o contexto, e essa lacuna de valor é distinta de `u`. Trocar o segundo por `unknown_type` perderia a identidade do domínio; trocar o terceiro por `opaque_type("unknown",1)` inventaria um domínio identificado.

## X-32 — Leitura preservada sem interpretar o domínio

```air
uncertainty u: TYPE_UNKNOWN scope={@x} origin=example:X-32/type
cell @x : unknown_type(u)
^entry:
  observe: opaque observedKind=example.observed_read
     knownOperands=[rx: VALUE_READ read(@x)] valueResults=[]
     memory={reads:[rx],other_reads:none,writes:[],other_writes:none}
     control={known:[normal(^done)],remainder:none}
     dependencies={known:[],remainder:none}
     reasons=[u]
```

O cenário estabelece uma observação de leitura pura e total de célula, sem outro efeito e sem uso de recursos. `rx` designa a mesma ocorrência no operando e no envelope, não duas leituras. A origem do uso é `example:X-32/operand:rx`; a declaração tem origem própria. `rx` conserva `unknown_type(u)`, `@x`, sua célula e a razão aplicável ao estado de entrada. A continuação é conhecida e nenhuma escrita foi introduzida. Essa operação neutra em relação ao domínio preserva a leitura; `nop` não seria equivalente.

## X-33 — Valor desconhecido com e sem domínio conhecido

As expressões seguintes são observadas como ocorrências distintas `VALUE_READ` de uma operação `opaque`, com continuação normal para `^done`, nenhuma escrita/uso de recurso e apenas as leituras explicitadas nas dependências.

```air
uncertainty u: TYPE_UNKNOWN scope={@x, expression:q} origin=example:X-33/type
uncertainty v: EXTERNAL_VALUE_UNKNOWN scope={expression:p,expression:q} origin=example:X-33/value
cell @x : unknown_type(u)
p = unknown(known(text), dependencies=[], remainingReads=none, reason=v)
q = unknown(unknown_type(u), dependencies=[read(@x)], remainingReads=none, reason=v)
```

`p` admite textos não determinados; `q` conserva domínio não estabelecido e a leitura de `@x`. Compartilhar `u` não afirma igualdade de domínio entre `q` e `@x`. `v` não substitui `u`. Nenhuma forma é literal, nem se torna igual a outra avaliação por reutilizar representação ou razão.

Outra variante tem resultado booleano e pureza estabelecidos pelo cenário, mas cálculo não interpretado:

```air
uncertainty u: TYPE_UNKNOWN scope={@x} origin=example:X-33/predicate-input
uncertainty r: PREDICATE_UNKNOWN scope={test} origin=example:X-33/predicate
cell @x : unknown_type(u)
^entry:
  test: branch unknown(known(bool), dependencies=[read(@x)], remainingReads=none, reason=r)
        then ^done else ^fault
```

O predicado tem tipo conhecido; sua dependência não. Os dois destinos e a leitura permanecem. Essa abstração não equivale a `branch read(@x)` e não infere `bool` de `@x`.

## X-34 — Precondições não são dispensadas

Cada linha da tabela é um caso positivo ou negativo independente, inserido em uma ocorrência de expressão/local apropriada. Considere `@x : unknown_type(u)`, `@y : unknown_type(u)`, `@n : known(int)` e `@s : known(text)`, com células independentes e `u` declarado como `TYPE_UNKNOWN` para `@x` e `@y`.

| Forma | Resultado obrigatório |
| --- | --- |
| `add(read(@x),int(1))` | `INVALID_IR`: não há domínio numérico conhecido para o primeiro argumento |
| `concat(read(@x),text("A"))` | `INVALID_IR`: não há `known(text)`/`known(bytes)` compatível |
| `not(read(@x))` | `INVALID_IR`: não há `known(bool)` |
| `eq(read(@x),read(@y))` | `INVALID_IR`: a lacuna compartilhada não prova domínio conhecido nem igualdade |
| `assign @n <- read(@x)` | `INVALID_IR`: origem sem domínio compatível conhecido |
| `assign @y <- read(@x)` | `INVALID_IR`: duas lacunas iguais não tornam a cópia tipada válida |
| `assign @x <- text("A")` | `INVALID_IR`: destino sem domínio compatível conhecido |
| `assign @s <- unknown(known(text))` | Válida: valor desconhecido de domínio `text` |
| `add(unknown(known(int)),int(1))` | Válida: valor não determinado, domínio numérico conhecido |

Uma construção observada de atribuição cujo destino e escrita completa estejam estabelecidos, mas cuja compatibilidade de tipo não esteja, pode ser preservada assim:

```air
uncertainty u: TYPE_UNKNOWN scope={@x,@y} origin=example:X-34/type
cell @x : unknown_type(u)
cell @y : unknown_type(u)
^entry:
  transfer: opaque observedKind=example.observed_assignment
     knownOperands=[rx: VALUE_READ read(@x),wy: VALUE_WRITE @y]
     valueResults=[@y]
     memory={reads:[rx],other_reads:none,writes:[wy],must_overwrite:[@y],other_writes:none}
     control={known:[normal(^done)],remainder:none}
     dependencies={known:[],remainder:none}
     reasons=[u]
```

O envelope não afirma cópia de identidade: conserva leitura de `@x`, escrita completa em `@y` e suas origens, admitindo conteúdo de destino desconhecido. Usar apenas `havoc.must @y` perderia a leitura. O cenário estabelece a sobrescrita e a continuação; a razão de tipo sozinha não provaria esses limites.

## X-35 — Escolhas preservam o conhecimento dos candidatos

Os locais abaixo são lidos em ocorrências distintas por uma operação como a de X-32, com os reads/endereço pertinentes no envelope.

```air
cell @s : known(text)
cell @n : known(int)
uncertainty u: TYPE_UNKNOWN scope={choice:mixed,choice:open} origin=example:X-35/type
mixed = choice([@s,@n], remainder=none, typeRef=unknown_type(u))
open = choice([@s], remainder=visible, typeRef=unknown_type(u))
```

O domínio do local selecionado em `mixed` não é único conhecido; os candidatos continuam respectivamente `known(text)` e `known(int)`. `read(mixed)` preserva ambos. Em `open`, não há garantia de domínio `text` para todo o restante visível. A anotação `known(text)` em qualquer dessas escolhas é inválida. Uma variante fechada contendo apenas `@s` usa `known(text)` e pode participar de `assign` compatível. `unknown_type` não cria uma conversão entre os candidatos nem torna uma escolha vazia fechada válida.

## X-36 — Assinatura parcial conserva argumento e resultado

O contrato do cenário estabelece um argumento por valor e um resultado normal, mas não estabelece seus domínios nem compatibilidade precisa. Seus efeitos no armazenamento do chamador são vazios além da escrita do resultado; admite retorno normal, exceção, término e divergência, sem outras transferências. São fatos contratuais com origem `example:X-36/contract`, não defaults.

```air
uncertainty u: TYPE_UNKNOWN scope={@x,parameter:1} origin=example:X-36/input-type
uncertainty v: TYPE_UNKNOWN scope={@y,result:1} origin=example:X-36/result-type
cell @x : unknown_type(u)
cell @y : unknown_type(v)
signature example.inspect: parameters=[value:unknown_type(u)] results=[unknown_type(v)]
^entry:
  k: invoke service literal("INSPECT")
     arguments=[value(read(@x))] results=[@y]
     contract=example.inspect effects=none normal ^done
```

`k` conserva o target literal, a ocorrência de leitura e os tipos desconhecidos das posições. No retorno normal, escreve conteúdo desconhecido em `@y`; na exceção, não escreve o resultado. A assinatura parcial não autoriza vinculação tipada precisa a um corpo nem igualdade de domínios por compartilhar `u`/`v`.

No contracaso independente em que o parâmetro exige `known(int)`, o mesmo argumento viola I-08: deve-se preservar a construção por `opaque` com seus fatos conhecidos, sem presumir inteiro. Igualmente, `return` preciso para resultado `known(text)` não aceita expressão de tipo desconhecido. Um resultado conhecido de domínio diferente do destino também não pode ser aceito por conversão implícita.
