# Exemplos — Memória, recursos e chamadas

**Informativo.** Usam o [contexto comum](00-notacao.md).

## X-11 — Dois objetos, uma célula

```air
cell @x : text
alias @y : text = @x
^entry:
  a: assign @x <- text("A")
  b: assign @y <- text("B")
  k: invoke program dynamic(read(@x))
```

`RD(@x,before(k))={b}` e `PV={"B"}`. A escrita nominal em @y afeta @x porque ambos designam a mesma célula. Uma análise por nome isolado erraria o target.

## X-12 — Destino entre duas células independentes

```air
cell @x : text
cell @y : text
^entry:
  a: assign @x <- text("A")
  d: assign @y <- text("D")
  b: assign choice([@x,@y], remainder=none, type=text) <- text("B")
  k: invoke program dynamic(read(@x))
```

A operação b escreve em uma alternativa admissível. `RD(@x,before(k))={a,b}`; `PV(@x)={"A","B"}`. Para @y, os candidatos são D e B. Não é permitido escolher o primeiro destino nem sobrescrever obrigatoriamente as duas células.

## X-13 — Escrita obrigatória versus escrita possível

As duas variantes abaixo são publicações independentes.

```air
cell @x : text
^entry:
  a: assign @x <- text("A")
  h: havoc.must @x reason EFFECT_UNKNOWN
  k: invoke program dynamic(read(@x))
```

Após escrita obrigatória: `RD={h}`, `PV={}; remainder=true`. A não é mantido como candidato corrente sustentado.

```air
cell @x : text
^entry:
  a: assign @x <- text("A")
  h: havoc.may {@x} reason EFFECT_UNKNOWN
  k: invoke program dynamic(read(@x))
```

Após escrita apenas possível: `RD={a,h}`, `PV={"A"}; remainder=true`. A alternativa sem escrita conserva A. O conjunto vazio da primeira variante não é resultado fechado.

## X-14 — Efeito limitado não apaga valor independente

```air
cell @target : text
cell @scratch : text
^entry:
  a: assign @target <- text("KNOWN")
  h: havoc.may {@scratch} reason EFFECT_UNKNOWN
  k: invoke program dynamic(read(@target))
```

As células são comprovadamente disjuntas no cenário. `PV(@target,before(k))={"KNOWN"}; remainder=false`. Ampliar artificialmente a incerteza de scratch para todo o programa perderia precisão sem necessidade; restringi-la sem prova também seria incorreto.

## X-15 — Parâmetro por bytes e alvo em uma fatia

```air
region @packet : bytes[12]
view @raw : bytes = @packet[0:12] codec=bytes.identity@1
view @action : text = @packet[0:3] codec=text.ascii@1
view @destination : text = @packet[4:8] codec=text.ascii@1
^entry:
  a: assign @action <- text("RUN")
  b: assign @destination <- fit_text(text("PGMA"),8," ")
  k: invoke service literal("ROUTER") action=call arguments=[reference(@raw)]
```

Antes de k, `PV(@destination)={"PGMA    "}; remainder=false`. O byte de offset 3 continua desconhecido; isso não invalida a fatia [4,12). O pacote completo não é conhecido.

Um consumidor que possui contrato de protocolo dizendo que a ação RUN interpreta [4,12) como nome com padding de espaço pode consultar a fatia e derivar PGMA. Sem esse contrato, a IR sustenta apenas chamada ao serviço ROUTER e o conteúdo do argumento; não inventa a dependência indireta. O protocolo não é embutido na IR.

## X-16 — Cópia de bytes sobrepostos

```air
region @buffer : bytes[6]
view @whole : bytes = @buffer[0:6] codec=bytes.identity@1
^entry:
  a: assign @whole <- bytes("ABCDEF")
  c: copy_bytes destination=@buffer[2:4] source=@buffer[0:4] length=4
  end: jump ^done
```

Após c, @whole contém `bytes("ABABCD")`. A origem foi capturada antes da atualização. A contribui para [0,2); c contribui para [2,6). O resultado não é produzido por leituras sucessivas dos bytes já alterados.

## X-17 — Nome literal não é identidade de catálogo

```air
^entry:
  k: invoke program literal("billing") namespace=example.program naming=exact
```

Há observação nominal de chamada ao nome exatamente `billing`. Esse fato independe de reaching definitions e do corpo da chamada. Não prova que `billing` e `BILLING` sejam o mesmo nome, que o artefato exista, que uma instância foi carregada ou que a interação retorne normalmente.

## X-18 — Argumento por referência e resultado normal

```air
cell @x : text
cell @status : int
^entry:
  a: assign @x <- text("A")
  k: invoke program literal("MUTATOR")
     arguments=[reference(@x)] results=[@status]
     effects={may_read(@x),may_write(@x)} normal ^next
^next:
  q: invoke program dynamic(read(@x))
```

`PV(@x,before(k))={"A"}`. No retorno normal, @x pode conservar A ou conter outro valor; @status recebe resultado desconhecido. Na saída excepcional, não há escrita automática do destino @status, a menos que um efeito contratual adicional a permita. `reference` sozinho não informa o efeito; o limite explícito é que admite alteração de x.

## X-19 — Estado persistente e múltiplas entradas

Este cenário substitui a entrada comum e declara duas entradas de uma unidade.

```air
cell @target : text lifetime=persistent
entry @fresh -> ^fresh state={@target:text("BOOT")} premise=FRESH_STATE
entry @resume -> ^resume state={@target:external_unknown}
^fresh:
  f: invoke program dynamic(read(@target))
^resume:
  r: invoke program dynamic(read(@target))
```

Na entrada fresh, o estado conhecido é uma premissa explícita dessa entrada: `PV(before(f))={"BOOT"}`. Em resume, o estado persistente anterior não é conhecido: `PV(before(r))={}; remainder=true`. A presença de valor inicial em uma declaração de origem não autoriza reaplicá-lo a resume. Uma consulta agregada entre entradas deve manter essa distinção de escopo.

## X-20 — Truncamento e arredondamento não são cópia de identidade

```air
cell @target : text
cell @amount : decimal
^entry:
  a: assign @target <- fit_text(text("ABCDEFGH"),4," ")
  n: assign @amount <- quantize(decimal(125,2),1,half_even)
  k: invoke program dynamic(read(@target))
```

`PV(@target,before(k))={"ABCD"}`. `decimal(125,2)` significa 1,25; em uma casa decimal com half-even, o resultado é 1,2. Trocar a por atribuição direta de ABCDEFGH ou usar arredondamento ambiental mudaria o comportamento. Nenhum tipo lógico pressupõe bytes ou codificação física.
