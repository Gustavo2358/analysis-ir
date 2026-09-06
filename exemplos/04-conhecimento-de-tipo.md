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
| `assign @n <- read(@x)` | `INVALID_IR`: não há prova de `sameDomain` entre origem e destino |
| `assign @y <- read(@x)` | `INVALID_IR`: duas lacunas iguais não tornam a cópia tipada válida |
| `assign @x <- text("A")` | `INVALID_IR`: não há prova de `sameDomain` entre destino e literal |
| `assign @x <- read(@x)` | Válida: a identidade da célula prova `sameDomain`, sem identificar seu domínio concreto |
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

O envelope não afirma cópia de identidade: conserva leitura de `@x`, escrita completa em `@y` e suas origens, admitindo conteúdo de destino desconhecido. Usar apenas `havoc.must @y` perderia a leitura. O cenário estabelece a sobrescrita e a continuação, mas não a compatibilidade ou cópia sem conversão; a razão de tipo sozinha não provaria esses limites. Se esses fatos estiverem estabelecidos, deve-se conservar `assign`, como em X-37.

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

No contracaso independente em que o parâmetro tem `known(int)` e não há prova de `sameDomain` com o argumento, a transmissão precisa viola I-08: deve-se preservar a construção por abstração com seus fatos conhecidos, sem presumir inteiro. Igualmente, `return` para resultado `known(text)` exige prova de mesmo domínio com a expressão. Domínios conhecidos distintos não podem ser conciliados por conversão implícita. X-38 demonstra a transmissão precisa quando o domínio concreto continua desconhecido, mas a relação está comprovada.

## X-37 — Cópia de domínio desconhecido comprovada

O cenário estabelece uma cópia sem conversão entre células independentes `@x` e `@y`, com domínio comum não identificado. `p` é uma premissa da entrada semântica do cenário, e não é inferida da operação `capture`.

```air
uncertainty u: TYPE_UNKNOWN scope={@x,@alias} origin=example:X-37/source-type
uncertainty v: TYPE_UNKNOWN scope={@y} origin=example:X-37/destination-type
cell @x : unknown_type(u)
alias @alias : unknown_type(u) = @x
cell @y : unknown_type(v)
premise p: sameDomain(@x,@y) scope=operation(capture)
           authority=example:X-37/input origin=example:X-37/domain-evidence
entry @main -> ^entry state={@x:external_unknown,@y:external_unknown}
^entry:
  self: assign @x <- read(@x)
  exact_alias: assign @alias <- read(@x)
  capture: assign @y <- read(@x)
  change: havoc.must @x reason EXTERNAL_VALUE_UNKNOWN
  observe: opaque observedKind=example.observed_read
     knownOperands=[ry: VALUE_READ read(@y)] valueResults=[]
     memory={reads:[ry],other_reads:none,writes:[],other_writes:none}
     control={known:[normal(^done)],remainder:none}
     dependencies={known:[],remainder:none}
     reasons=[v]
```

`self` é válido por identidade; `exact_alias`, pela associação exata; `capture`, por `p` e pela regra de domínio de `read`. As três operações conservam suas ocorrências e definições. Os `TypeRef` continuam desconhecidos. No comportamento em que a entrada fornece um valor `v₀`, `@y` após `capture` contém exatamente esse valor; a escrita posterior em `@x` não altera a captura. `v₀` é uma variável da explicação, não literal ou campo de IR. O consumidor não precisa enumerá-lo, mas conserva a relação com a avaliação em `before(capture)`.

Contracasos independentes: remover `p` invalida apenas a cópia entre células distintas; mudar seu escopo para `operation(change)` também, pois não contém `operation_site(capture)`. Manter apenas a mesma lacuna nos dois objetos não substitui a prova. Acrescentar `add(read(@x),read(@y))`, `concat(read(@x),read(@y))`, `not(read(@x))` ou `eq(read(@x),read(@y))` continua inválido mesmo com `p`. Uma cadeia de premissas que una `known(int)` a `known(text)` através de `@x` é contraditória.

## X-38 — Transmissão sem conversão através de assinatura

Esta publicação substitui o contexto de unidade única. O chamador tem células independentes `@source : unknown_type(u)` e `@result : unknown_type(v)`, com estado externo desconhecido. A entrada `e` do chamado tem um parâmetro por valor de `unknown_type(w)`, objeto de entrada `@param : unknown_type(w)` associado a célula própria e uma posição de resultado `unknown_type(z)`. As quatro lacunas são declaradas com código `TYPE_UNKNOWN` e origens próprias do cenário.

O contrato de transmissão estabelece, com premissas distintas, autoridade e origem `example:X-38/contract`, os vínculos abaixo. As posições da declaração de `e` distinguem-se de suas instâncias na fronteira de `k`, conforme 02, §1.4. `@source`/`@result` pertencem ao chamador; `@param` pertence ao chamado. Cada premissa vale em todas as execuções dos sites denotados por seu escopo, não em uma única ativação selecionada.

| Premissa | Sujeitos | Escopo |
| --- | --- | --- |
| `p_in` | `sameDomain(@source, parameter(k,e,1))` | `invocation(k)` |
| `p_binding` | `sameDomain(parameter(e,1), @param)` | `entry(e)` |
| `p_return` | `sameDomain(@param, result(e,1))` | `operation(r)` |
| `p_out` | `sameDomain(result(k,e,1), @result)` | `invocation(k)` |

```air
unit @caller entry ^entry
  ^entry:
    k: invoke target=internal(e) action=call
       arguments=[value(read(@source))] results=[@result]
       effects=none contract=example.identity_transfer normal ^done
  ^done:
    done: halt normal

unit @callee entry e -> ^body
  state={@param:parameter(1)}
  ^body:
    r: return [read(@param)]
```

O cenário estabelece ausência de efeitos no armazenamento do chamador além do resultado e retorno normal sem exceção, término ou divergência. O corpo mostra que o valor recebido é devolvido; `sameDomain` sozinho não daria essa informação. Se a captura de `@source` fornece `v₀`, o parâmetro e o resultado transmitido carregam `v₀`; os domínios concretos permanecem não identificados. Substituir `value` por `copy` conserva a transmissão de valor, com armazenamento privado do chamado. Uma variante por `reference` exige também a associação precisa do local e não implica cópia do conteúdo.

Remover qualquer prova impede a transmissão precisa correspondente. `p_binding` não pode validar `r`, porque `entry(e)` não inclui `operation_site(r)`; `p_in`/`p_out` também não valem no corpo. Reutilizar `p_in`/`p_out` numa chamada diferente não é autorizado por igualdade de nomes das posições. Uma segunda chamada exige seus vínculos aplicáveis, mas pode usar a mesma declaração de entrada/corpo com `p_binding` e `p_return` universais nos respectivos sites. Repetir `k`, inclusive recursivamente, reaplica seus vínculos aos papéis daquela execução e não iguala os valores de ativações diferentes. Se o chamado devolver outro valor de mesmo domínio, o resultado muda conforme o corpo: igualdade de domínio não promete função identidade.

## X-39 — Domínio comum em escolhas sem tipo concreto

As células `@x`, `@y` e `@dst` têm lacunas de tipo distintas, com origem no cenário. Duas premissas com autoridade `example:X-39/input`, origens próprias e `scope=operation(c)` comprovam `sameDomain(@x,@dst)` e `sameDomain(@y,@dst)`. A construção observada copia o valor selecionado sem conversão.

```air
^entry:
  c: assign @dst <- read(choice([@x,@y], remainder=none, typeRef=unknown_type(u_choice)))
  end: jump ^done
```

`u_choice` é uma lacuna `TYPE_UNKNOWN` própria da escolha. A cópia é válida porque ambos os candidatos têm o mesmo domínio do destino, embora nenhum domínio concreto tenha sido identificado. As duas alternativas de valor e suas ocorrências continuam observáveis. A prova não escolhe uma delas.

Abrir `remainder=visible` sem garantia de mesmo domínio para todo esse restante invalida a cópia precisa; provar só o primeiro candidato também não basta. Na variante aberta válida, `pc` é o `OperandId` do local `choice` dentro de `read`, e acrescenta-se:

```air
premise p_open: sameDomain(pc,@dst) scope=operation(c)
                authority=example:X-39/open-domain origin=example:X-39/remainder-evidence
```

A autoridade do cenário estabelece que todos os locais possíveis de `pc`, inclusive todo membro do restante visível, têm o domínio de `@dst`. A premissa sobre `pc` cobre a escolha inteira e basta, mesmo sem as duas premissas individuais. As alternativas não desaparecem e `u_choice` continua desconhecido; o exemplo não afirma essa garantia para memória fora da escolha. Remover `p_open` mantendo apenas as provas dos candidatos volta a invalidar a variante aberta. Mover a premissa para outra operação também a torna inaplicável. Substituir `pc` por outra ocorrência, ainda que copie a mesma expressão, não prova a cobertura desta. Se `@dst` for `known(text)` e um candidato for `known(int)`, a premissa universal é contraditória.

Numa variante de destino `choice([@x,@y],...)`, a escrita permanece alternativa/fraca, mesmo com domínio comum comprovado. Duas avaliações independentes de uma escolha entre `known(int)` e `known(text)` não têm `sameDomain` apenas porque reutilizam a mesma expressão ou lacuna.

## X-40 — Aplicabilidade e interseção de escopos

Esta publicação tem unidades distintas `U` e `V`, entradas `eU` e `eV` respectivamente, e células visíveis `@a`, `@b`, `@c` com tipos desconhecidos de lacunas próprias. `U` contém as operações distintas `copy` (`assign @c <- read(@a)`) e `other` (`nop`), além de `invoke k` para `eV` e outra chamada `k2`. `V` contém `return r`. As demais condições de memória, controle e assinatura são satisfeitas independentemente das premissas testadas.

`p1: sameDomain(@a,@b)` e `p2: sameDomain(@b,@c)` têm IDs distintos, autoridade `example:X-40/input` e origens próprias; são as únicas bases não reflexivas disponíveis para a cópia. Cada linha abaixo é uma variante independente:

| `p1.scope` | `p2.scope` | Interseção | Prova em `operation_site(copy)` |
| --- | --- | --- | --- |
| `publication` | `operation(copy)` | `operation(copy)` | Aplicável |
| `unit(U)` | `operation(copy)` | `operation(copy)` | Aplicável |
| `operation(copy)` | `operation(other)` | Vazia | Inaplicável |
| `unit(V)` | `operation(copy)` | Vazia | Inaplicável |
| `entry(eU)` | `operation(copy)` | Vazia | Inaplicável, mesmo se `copy` for a primeira operação de `eU` |
| `intersection(unit(U),operation(copy))` | `publication` | `operation(copy)` | Aplicável |

Nas variantes aplicáveis, a regra de `read` permite concluir `sameDomain(@c,read(@a))` e a cópia é válida. Nas demais, a cópia precisa é inválida por falta de prova, não por falta de tipo concreto. A interseção vazia é um escopo bem formado sem sites, não uma licença para usar um escopo maior.

Para uma premissa com sujeitos apropriados à chamada, `intersection(operation(k),invocation(k))` cobre apenas `invocation_site(k)`: não cobre `operation_site(k)`, `entry_site(eV)`, `operation_site(r)` ou `invocation_site(k2)`. `unit(U)` inclui ambos os sites de `k`, mas não os sites de `V`; contenção de uma unidade tampouco transfere essa cobertura. `publication` inclui todos esses sites sem identificar seus sujeitos entre si. Repetição/recursão reaplica os fatos universalmente sem identificar ativações; nenhum escopo promete alcançabilidade.

Contracasos de formação: `invocation(other)`, `unit` referindo um `EntryId`, referência a ID inexistente, forma `activation(U)` e composição circular/não finita são inválidos, mesmo em premissa não utilizada. Uma condição textual de caminho ou uma garantia apenas da primeira execução de `k` não pode substituir `DomainProofScope`.

## X-41 — Contrato externo materializado no site

A unidade possui células independentes `@x : unknown_type(u)` e `@y : unknown_type(v)`, com lacunas e origens próprias. A autoridade `example.inspect`, versão `rev-1`, estabelece uma posição por valor e um resultado normal, de domínios não identificados. Seus efeitos são vazios além do resultado; admite retorno, exceção, término e divergência. O cenário não fornece `sameDomain` entre argumento/parâmetro ou resultado/destino, portanto usa a transmissão conservadora de 04, §7.1.

```air
origin c: CONTRACT authority=example.inspect version=rev-1
^entry:
  k: invoke service literal("INSPECT") action=call
     arguments=[value(read(@x))] results=[@y]
     signature=external(
       parameters={known:[1:value:unknown_type(u)],remainder:none},
       results={known:[1:unknown_type(v)],remainder:none})
     contract={authority:example.inspect,version:rev-1,evidence:[c]}
     effects=none
     outcomes={normal(^done),any_exception(^fault),halt,diverge; remainder=none}
```

A aridade é conhecida apesar dos tipos desconhecidos. `external_parameter(k,1)` e `external_result(k,1)` identificam essas posições; repetir a razão `u`/`v` nos objetos não cria prova. As origens das posições e dos fatos são contratuais do cenário; `c` explica a autoridade, enquanto a assinatura, os limites e os outcomes fornecem a semântica, sem consultar `example.inspect`. Não existe declaração `Contract` top-level.

Variantes independentes: assinatura indisponível usa `known:[]` com `remainder=unknown(g)` em cada direção, preservando os argumentos/resultados observados e as garantias da interação. As lacunas `g` são declaradas por direção com código `CONTRACT_UNKNOWN` e origem da indisponibilidade. Autoridade desconhecida usa `contract=unknown(c_unknown)`, também com lacuna `CONTRACT_UNKNOWN` declarada; não muda fatos sustentados por outra evidência. Acrescentar `k2` com a mesma autoridade não reutiliza sujeitos/provas limitados a `k`. Uma premissa de transmissão precisa em `k` deve identificar suas ocorrências/posições e ter escopo `invocation(k)`; não nasce do nome INSPECT.

Como contracasos, remover a origem `c`, referir uma posição inexistente ou usar `ResourceId` como target viola fechamento/modelo. Alterar o contrato da autoridade exige nova publicação com fatos revalidados, não atualização preguiçosa desta publicação imutável.
