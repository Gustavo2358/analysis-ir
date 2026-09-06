# Notação de leitura dos exemplos

**Analysis IR 2.0.0 — Informativo**

## 1. Finalidade

Os blocos `air` ilustram a sintaxe abstrata da especificação. Não definem um formato de intercâmbio, parser, linguagem de implementação ou API. Todos os seus elementos têm significado nos documentos normativos. A notação omite repetição de metadados, não fatos semânticos necessários aos resultados esperados.

Cada cenário X-01 a X-36 é uma publicação independente. Variantes dentro do mesmo cenário são explicitamente independentes quando indicado. As consultas escritas após os blocos são resultados de consumers, **não conteúdo da IR**.

## 2. Identificadores e abreviações

| Notação | Significado |
| --- | --- |
| `@x` | Identidade de objeto no namespace da unidade corrente |
| `^entry` | Label de sequência |
| `a: assign ...` | Operação com identidade local `a` |
| `read(@x)` | `read(object(@x))` |
| `text("A")`, `int(1)`, `bool(true)` | Literais tipados |
| `bytes("ABC")` | Octetos ASCII explicitamente indicados, não codificação ambiental |
| `@r[2:4]` | Intervalo de 4 octetos a partir do offset 2; não fim no índice 4 |
| `literal("P")` | Nome externo literal da categoria e namespace indicados |
| `dynamic(expr)` | Nome externo calculado por expressão de tipo `text` |
| `unknown(R)` | Valor desconhecido com `R: TypeRef`, sem escritas/controle oculto |
| `known(T)` | Domínio de valor estabelecido; `T` isolado abrevia essa forma em posição de `TypeRef` |
| `unknown_type(u)` | Domínio não estabelecido, referindo a lacuna `TYPE_UNKNOWN` de identidade `u` |
| `before(k)` | Ponto imediatamente antes da operação `k` |
| `RD(@x,before(k))` | Consulta derivada de definições alcançáveis |
| `PV(@x,before(k))` | Consulta derivada de valores possíveis |

Todo uso abreviado de `unknown(R)` tem razão de valor `SOURCE_SEMANTICS_UNAVAILABLE`, lista de leituras vazia e `remainingReads=none`, salvo dependências explicitamente fornecidas. `unknown(T)` com `T: Type` abrevia `unknown(known(T),...)`. Isso é valor não determinado, não ausência de expressão. Se `R=unknown_type(u)`, a lacuna de tipo é adicional à razão de valor. Identidades de lacunas e origens omitidas pela abreviação são próprias da ocorrência/cenário.

## 3. Declarações de células

`cell @x : R` abrevia uma declaração nominal e sua célula própria com o mesmo `TypeRef=R` e duração `activation`. `cell @x : T` com `T: Type` usa `known(T)`. **Nos exemplos que a usam**, a independência em relação às demais células é uma premissa expressa do cenário. Ela não pode ser generalizada para declarações reais apenas por nomes diferentes.

`alias @y : T = @x` associa outro objeto ao mesmo armazenamento. `region @r : bytes[n]` declara região de n octetos. `view @v : T = @r[o:n] codec=C` declara vista com codec explícito. Toda associação/intervalo mostrado é conhecido, salvo marcação contrária. Nomes de portas utilizados em `local.boundary` abreviam suas declarações no namespace da unidade; as referências em `completion` apontam para essas declarações.

O estado de entrada de células não inicializadas é desconhecido, com definição de entrada `ENTRY(@x)`. `ENTRY` é identidade do produto de análise, não operação da IR. Valores persistentes e múltiplas entradas são explicitados nos cenários que os usam.

## 4. Contexto comum completo

Cada fragmento representa o corpo de uma unidade `@main`, com entrada `^entry`, assinatura sem parâmetros/resultados e armazenamento declarado no próprio exemplo. O contexto fornece também estas sequências terminais, salvo substituição explícita:

```air
^done:
  done: halt normal
^fault:
  fault: halt abnormal
```

Toda operação possui origem derivada do cenário e de sua identidade, por exemplo `example:X-02/operation:b`. Não se inventa span de um programa externo. Inventário é completo, exceto quando o cenário especifica parcialidade. Premissas e envelopes mostrados são fatos do cenário.

## 5. Contexto comum de interações

A abreviação de interação abaixo é usada nos exemplos:

```air
k: invoke program dynamic(read(@target))
```

Ela significa ação `call`, categoria `program`, namespace `example.program`, interpretação do nome `exact`, argumentos e resultados vazios, contrato externo desconhecido, efeitos `may_read(visible)` e `may_write(visible)` e os outcomes:

```text
normal(^done), any_exception(^fault), halt, diverge; remainder=none
```

O cenário assume que a forma de interação exclui saltos arbitrários para o interior do chamador; não assume que retorna sempre. `visible` inclui armazenamento da unidade potencialmente acessível e ambiente externo. Essa hipótese de controle deve ser ampliada quando não puder ser sustentada por um produtor.

`normal ^next` substitui apenas a continuação normal. `effects=none [C-PURE]` identifica contrato explícito de ausência de efeitos no armazenamento do chamador/compartilhado; não é default. `effects=may_write(@x)` restringe escritas a x, declara outras escritas vazias e mantém somente as leituras explicitadas ou dos operandos. Um contrato restritivo é premissa do cenário, não conclusão de análise.

Para outras categorias, o namespace é `example.<categoria>` e a ação deve ser indicada. A notação `results=[@x]` usa assinatura externa de mesmo domínio conhecido de x; seus resultados só são escritos no retorno normal. Cenários com `unknown_type` DEVEM explicitar a assinatura parcial e o contrato conservador, sem herdar uma compatibilidade presumida dessa abreviação.

## 6. Como interpretar os resultados

`PV = {"B", "C"}; remainder=false` é enumeração fechada no modelo do cenário. Não é afirmação de que os dois caminhos sempre executam. `PV = {"B"}; remainder=true` conserva um candidato enumerado e admite outros valores.

Resultados RD/PV são anteriores à interação consultada; os efeitos dessa própria interação não contaminam retroativamente seu target. Prefixos como `ENTRY` e `UNKNOWN_WRITE` são identificadores derivados de análise.

Os exemplos não transportam GEN/KILL, RD, PV ou CFG calculado. Apresentá-los em texto separado demonstra o que a representação deve permitir aos consumidores produzir.
