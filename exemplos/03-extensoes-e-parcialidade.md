# Exemplos — Extensões, controle e parcialidade

**Informativo.** Usam o [contexto comum](00-notacao.md), exceto onde indicado.

## X-21 — Recursos de espécies diferentes

```air
cell @table : text
^entry:
  a: assign @table <- text("ACCOUNT")
  read: invoke table dynamic(read(@table)) action=read
        arguments=[] effects=none [C-PURE] normal ^service
^service:
  s: invoke service literal("AUDIT") action=call
```

O inventário contém acesso de leitura à tabela ACCOUNT e chamada ao serviço AUDIT. Não são duas chamadas a programa. A primeira depende do valor de @table no ponto de read; a segunda é literal. O contrato C-PURE limita memória local, não nega a interação com o recurso.

## X-22 — Destino indireto mutável

```air
capability control.indirect@1
cell @route : label({^p,^q})
^entry:
  a: assign @route <- label(^p,{^p,^q})
  b: assign @route <- label(^q,{^p,^q})
  j: indirect.jump read(@route) within {^p,^q}
^p:
  p: invoke program literal("P")
^q:
  q: invoke program literal("Q")
```

O CFG conservador inicial admite p e q por causa do universo de tipo declarado. Com efeitos e valores suficientes, a análise de @route antes de j encontra apenas q. Uma revisão refinada pode remover p como successor possível de j, preservando a observação literal de P no inventário. Nenhum destino é inferido por correspondência textual de nome de label.

## X-23 — Duas invocações locais do mesmo trecho

```air
capability control.local@1
cell @x : text
^entry:
  a: assign @x <- text("A")
  c1: local.invoke ^sub completion={end_sub} resume ^after_first
^after_first:
  first: invoke program dynamic(read(@x)) effects=none [C-PURE] normal ^second_call
^second_call:
  c2: local.invoke ^sub completion={end_sub} resume ^after_second
^after_second:
  second: invoke program dynamic(read(@x))
^sub:
  b: assign @x <- text("B")
  finish: local.boundary end_sub default ^fault
```

O retorno associado a c1 vai a after_first; o associado a c2 vai a after_second. Ambos observam B. Uma aresta de retorno de finish diretamente a after_second durante c1 seria espúria. A chamada local não reinicializa @x nem cria cópia privada do estado.

## X-24 — Trechos que compartilham operações e portas de conclusão

```air
capability control.local@1
cell @x : text
cell @short_result : text
^entry:
  short: local.invoke ^part_one completion={middle} resume ^after_short
^after_short:
  save: assign @short_result <- read(@x)
  long: local.invoke ^part_one completion={end_range} resume ^after_long
^part_one:
  b: assign @x <- text("B")
  mid: local.boundary middle default ^part_two
^part_two:
  c: assign @x <- text("C")
  end: local.boundary end_range default ^done
^after_long:
  k: invoke program dynamic(read(@x))
```

Na invocação short, middle coincide com a porta do topo e retorna após B. Na invocação long, middle não coincide; segue ao default, executa C e retorna por end_range. Antes de k, @short_result contém B e @x contém C.

Uma entrada ordinária em part_one com pilha local vazia passaria pelas duas portas por seus defaults. As operações compartilhadas não precisam ser duplicadas. A regra de topo é explícita e não equivale a buscar qualquer frame contendo a porta.

## X-25 — Controle desconhecido não vira fallthrough

```air
cell @target : text
^entry:
  a: assign @target <- text("A")
  u: opaque observedKind=sample.uninterpreted_transfer
     knownOperands=[] valueResults=[]
     memory={other_reads:visible,other_writes:visible}
     control={known:[normal(^join)],remainder:any_control}
     dependencies={known:[],remainder:any_resource}
     reason=CONTROL_UNKNOWN
^join:
  k: invoke program dynamic(read(@target))
```

A continuação conhecida é conservada, mas não é a única possibilidade. O CFG inclui fronteira aberta; consultas influenciadas por u não podem alegar enumeração exaustiva de caminhos, valores ou dependências. Não se pode simplesmente conectar u a join e fechar o grafo. A observação da própria k continua no inventário.

## X-26 — Extensão desconhecida com fallback tipado

```air
capability sample.normalize_name@1
cell @target : text
^entry:
  a: assign @target <- text("Alpha")
  x: extension sample.normalize_name@1 input=read(@target) output=@target
     fallback.memory={reads:[@target],must_overwrite:[@target],other_writes:none}
     fallback.control={known:[normal(^next)],remainder:none}
     fallback.dependencies={known:[],remainder:none}
^next:
  k: invoke program dynamic(read(@target))
```

O contrato ilustrativo dessa extensão deve definir seu payload e regra precisa antes de ser adotado. Um consumidor que desconhece a regra pode usar o envelope: x certamente sobrescreve target por resultado não interpretado e segue a next. Ele NÃO deve conservar Alpha como valor corrente exato nem executar a extensão e seu fallback como eventos distintos.

O exemplo demonstra consumo conservador de extensão; não registra `sample.normalize_name` como capacidade padrão V1. Para consumidores precisos, a sua especificação externa é requerida.

## X-27 — Identidades locais iguais em unidades diferentes

```air
unit @first entry ^entry
  cell @x : text
  ^entry:
    a: assign @x <- text("FIRST")
    k: invoke program dynamic(read(@x))

unit @second entry ^entry
  cell @x : text
  ^entry:
    a: assign @x <- text("SECOND")
    k: invoke program dynamic(read(@x))
```

As identidades completas incluem a unidade. Na primeira k, o target é FIRST; na segunda, SECOND. O contexto comum fornece saídas separadas para cada unidade. Igualdade dos textos locais `@x`, `a` ou `k` não une células, definições ou pontos.

## X-28 — Input ausente e observação literal independente

```air
inventory partial
^entry:
  missing: opaque observedKind=sample.missing_region
     knownOperands=[] valueResults=[]
     memory={other_reads:visible,other_writes:visible}
     control={known:[normal(^known)],remainder:any_control}
     dependencies={known:[],remainder:any_resource}
     reason=INPUT_MISSING
^known:
  k: invoke program literal("KNOWN")
```

O site literal KNOWN pode ser publicado como `OBSERVED`. A publicação não está completa, e o envelope impede afirmar que KNOWN é a única dependência ou que k necessariamente executa. Nenhuma inferência sobre o conteúdo faltante é necessária para conservar a observação literal escrita na IR.

## X-29 — Valores infinitos e limite honesto

```air
cell @target : text
cell @again : bool
^entry:
  a: assign @target <- text("P")
  start: jump ^test
^test:
  input: havoc.must @again reason EXTERNAL_VALUE_UNKNOWN
  t: branch read(@again) then ^body else ^join
^body:
  grow: assign @target <- concat(read(@target),text("X"))
  back: jump ^test
^join:
  k: invoke program dynamic(read(@target))
```

O conjunto de valores admite P, PX, PXX e assim por diante. Uma análise finita pode publicar, por exemplo, `{P,PX}; remainder=true`, com razão de aproximação/limite. Não pode publicar apenas `{P,PX}; remainder=false`. O inventário de operações continua completo; o limite incide no resultado de análise, não autoriza remover grow ou alguma iteração semântica.

## X-30 — Relação estrutural não é interação executável

```air
artifact @source
artifact @definitions
artifact_relation inc: @source includes @definitions
artifact_relation sch: @source uses_schema external(schema,"CUSTOMER-SCHEMA")

^entry:
  k: invoke table literal("CUSTOMER") action=read
```

O consumer pode emitir duas relações estruturais e uma observação executável de leitura de tabela. A relação uses_schema não prova, por si só, leitura de todas as tabelas descritas pelo schema. Includes não cria aresta no CFG. A relação executável CUSTOMER tem seu próprio site, ação e namespace.
