# 02 — Tipos, valores, referências e avaliação

**Analysis IR 1.0.0 — Normativo**

## 1. Tipos de valor do núcleo

| Tipo | Domínio e igualdade |
| --- | --- |
| `bool` | `true` e `false` |
| `int` | Inteiros matemáticos, sem overflow implícito |
| `decimal` | Valores exatos `m × 10^-s`, com `m` inteiro e `s ≥ 0`; igualdade numérica |
| `text` | Sequência finita de valores escalares Unicode; igualdade elemento a elemento |
| `bytes` | Sequência finita de octetos, cada um em `0..255` |
| `opaque_type(id, version)` | Domínio definido por extensão; não possui operações precisas implícitas |

`label(scope)` é acrescentado por `control.indirect@1`. Endereços numéricos e rótulos de controle não são `int` intercambiáveis. Uma sequência de bytes não é texto sem conversão explícita. Tipo de valor não determina tamanho, representação física, alinhamento, codificação ou duração de armazenamento.

Nenhuma comparação de `text` remove espaços, muda caixa, aplica normalização Unicode ou usa collation ambiental implicitamente. Comportamento diferente exige operação definida ou extensão. Floats, decimal de precisão limitada e inteiros modulares não podem ser aproximados por aritmética exata sem declarar a abstração.

## 2. Literais

Um literal tem tipo e valor. Sua grafia-fonte pertence à origem. Strings vazias, zeros, sinais e bytes `00` são valores válidos e NÃO DEVEM ser confundidos com ausência. Literais de `decimal` podem ter várias grafias para o mesmo valor; comparação não depende da grafia.

Um literal de recurso é um nome no namespace declarado, não prova de carregamento, existência ou execução do recurso.

## 3. Expressões

Expressões do núcleo são puras: não escrevem memória, não invocam código, não transferem controle e não lançam exceções. Podem ler estado através de `read`. São avaliadas no ponto de uso, no estado anterior à operação. Compartilhar sua representação NÃO as transforma em valores previamente capturados.

| Forma | Assinatura e semântica |
| --- | --- |
| `literal(T,v)` | Valor `v` do tipo `T` |
| `read(place)` | Valor atual do local designado; tipo declarado da vista |
| `unknown(T, dependencies, remainingReads, reason)` | Valor não determinado de `T`, com dependências e lacuna explícitas |
| `eq(a,b)`, `ne(a,b)` | Mesmo tipo conhecido; igualdade definida nesse domínio |
| `lt(a,b)`, `le(a,b)`, `gt(a,b)`, `ge(a,b)` | `int` ou `decimal` do mesmo tipo; ordem matemática |
| `not(a)`, `and(a,b)`, `or(a,b)` | Booleanos; operadores puros e estritos |
| `add(a,b)`, `sub(a,b)`, `mul(a,b)`, `neg(a)` | `int` ou `decimal`; resultado exato no mesmo domínio |
| `to_decimal(a)` | Conversão exata de `int` para `decimal` |
| `quantize(a,s,mode)` | Decimal arredondado a `s ≥ 0` casas; `toward_zero` ou `half_even` |
| `concat(a,b)` | Concatenação de dois `text` ou dois `bytes` |
| `length(a)` | Número de elementos de `text` ou `bytes`; resultado `int` |
| `fit_text(a,n,pad)` | Texto com exatamente `n ≥ 0` caracteres: trunca à direita ou preenche à direita com o único caractere `pad` |
| `slice_text(a,start,count)` | Subsequência de índices `[start,start+count)`; exige `start ≥ 0`, `count ≥ 0` e limite dentro de `a` |
| `trim_right(a,chars)` | Remove o maior sufixo de caracteres pertencentes ao conjunto explícito `chars` |

Os parâmetros `s`, `n` e `pad` são valores constantes nesta V1. `slice_text` só é admissível como expressão pura se a validade do intervalo estiver assegurada por fatos ou pelo domínio de entrada explicitado. Caso contrário, o produtor DEVE materializar teste/saída excepcional ou operação opaca com envelope apropriado; não pode inventar clipping ou ignorar erro possível.

As operações numéricas não têm conversões implícitas. `half_even` escolhe o valor mais próximo e, no empate, o coeficiente inteiro par na escala de destino. `toward_zero` descarta a fração excedente em direção a zero. Divisão geral, truncamento de precisão, representação binária e ordenação de texto não são operações exatas do núcleo.

## 4. Desconhecimento não é constante

`unknown(T,...)` não é uma constante com uma grafia especial. Avaliações distintas não têm igualdade ou correlação garantida. A identidade de uma expressão desconhecida não autoriza concluir `eq(unknown,unknown)=true`.

As dependências conhecidas de um valor desconhecido DEVEM ser preservadas. `remainingReads=none` afirma que a lista de leituras é completa; `remainingReads=scope` admite outras leituras dentro do escopo. Isso não permite escritas ou controle oculto. Uma construção cujo efeito pode ser impuro DEVE ser uma operação, não `unknown` dentro de uma expressão.

Um predicado puro desconhecido pode alimentar `branch`: os dois destinos continuam visíveis. Falta de semântica do predicado não equivale, por si só, a desconhecimento da estrutura de controle.

## 5. Referências e locais

Um `Place` descreve onde um valor é lido ou escrito. Formas V1:

```text
object(ObjectId)
choice(candidates: Place[], remainder: none | MemoryScope, type: Type)
region_slice(RegionId, offset, length, codec)          [memory.regions@1]
```

`object` designa uma entidade já identificada, não inicia lookup por nome. `choice` preserva alternativas ainda não selecionáveis; uma ocorrência não pode escolher a primeira. `remainder=none` declara alternativas exaustivas. Lista vazia com `remainder=none` não designa local algum e é inválida para `read` ou `assign`.

O `type` de `choice` DEVE ser compatível com todas as alternativas; se isso não puder ser assegurado, a operação deve usar representação opaca apropriada. Uma incerteza nominal não deve ser normalizada para certeza apenas porque todas as alternativas têm a mesma grafia.

`MemoryScope` é uma união tipada de objetos, regiões, células, armazenamento visível à unidade ou todo o armazenamento da publicação/ambiente. Toda referência aberta DEVE incluir o restante externo potencial quando ele não puder ser excluído. Escopo textual como “outros dados” sem significado de conjunto é insuficiente.

## 6. Papéis e identidade de operandos

Cada ocorrência de operando possui identidade, posição, tipo e papel. Os papéis padrão são `VALUE_READ`, `VALUE_WRITE`, `ADDRESS_READ`, `PREDICATE`, `CALL_TARGET`, `ARGUMENT_VALUE`, `ARGUMENT_REFERENCE`, `RESULT_TARGET`, `RESOURCE_TARGET` e `CONTROL_TARGET`.

Papel não substitui semântica da operação. Um argumento por referência não prova que o conteúdo é lido ou escrito. Um alvo de chamada calculado é uma leitura de valor mesmo que seu papel seja `CALL_TARGET`. Calcular o índice de um destino também é leitura; a atualização do destino não apaga essa leitura anterior.

Uma expressão composta permite derivar recursivamente suas leituras. O mesmo objeto lido duas vezes pode ter duas ocorrências. Os consumidores podem agregar efeitos, mas devem manter correlação suficiente para explicar cada uso.

## 7. Captura e ordem

Em `assign(dst, value)`, o endereço de `dst` e `value` são determinados no mesmo estado anterior; a escrita ocorre depois. Uma atribuição de valor NÃO equivale a guardar uma expressão para reavaliar no futuro.

Quando uma construção-fonte avalia uma expressão uma única vez e reutiliza o resultado após atualizações, o produtor DEVE capturá-lo em objeto auxiliar próprio ou usar extensão com semântica equivalente. Esse objeto tem identidade e origem derivada, não é um símbolo-fonte inventado.

Operadores booleanos puros são estritos. Short-circuit com operações impuras DEVE ser explicitado por controle. Não é permitido esconder chamadas em operandos de `and` ou `or`.
