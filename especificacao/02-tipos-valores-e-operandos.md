# 02 — Tipos, valores, referências e avaliação

**Analysis IR 2.0.0 — Normativo**

## 1. Tipos de valor do núcleo

`Type` identifica um domínio semântico de valores. Não descreve quanto se sabe sobre esse domínio.

| Tipo | Domínio e igualdade |
| --- | --- |
| `bool` | `true` e `false` |
| `int` | Inteiros matemáticos, sem overflow implícito |
| `decimal` | Valores exatos `m × 10^-s`, com `m` inteiro e `s ≥ 0`; igualdade numérica |
| `text` | Sequência finita de valores escalares Unicode; igualdade elemento a elemento |
| `bytes` | Sequência finita de octetos, cada um em `0..255` |
| `opaque_type(id, version)` | Domínio conhecido, identificado e definido por extensão; não possui operações precisas implícitas |

`label(scope)` é acrescentado por `control.indirect@1`. Endereços numéricos e rótulos de controle não são `int` intercambiáveis. Uma sequência de bytes não é texto sem conversão explícita. Tipo de valor não determina tamanho, representação física, alinhamento, codificação ou duração de armazenamento.

Nenhuma comparação de `text` remove espaços, muda caixa, aplica normalização Unicode ou usa collation ambiental implicitamente. Comportamento diferente exige operação definida ou extensão. Floats, decimal de precisão limitada e inteiros modulares não podem ser aproximados por aritmética exata sem declarar a abstração.

### 1.1 Conhecimento de tipo

Todo ponto que carrega conhecimento do domínio de um valor usa `TypeRef`:

```text
TypeRef = known(Type) | unknown_type(UncertaintyId)
```

`known(T)` afirma que o domínio é `T`. `unknown_type(u)` afirma que o domínio não foi estabelecido e DEVE referir uma lacuna existente de código `TYPE_UNKNOWN`, com escopo e proveniência. Não é um novo domínio de valores, um tipo universal, uma variável de unificação nem uma permissão de conversão. A repetição de `u` compartilha a explicação da lacuna; NÃO prova igualdade de domínios entre entidades distintas, nem igualdade de valores.

`known(opaque_type(id, version))` identifica um domínio mesmo quando o consumidor não interpreta sua extensão. Essa falta de suporte é tratada pela negociação de capacidades, não por substituição por `unknown_type`. `opaque_type` NÃO DEVE representar tipo desconhecido, inclusive por identificadores como `"unknown"`. Tipos de extensões padronizadas, como `label(S)`, também usam `known(label(S))` quando estabelecidos.

`Object`, `Cell`, `Place`, resultados de expressões, ocorrências de operandos e posições de parâmetros/resultados em assinaturas DEVEM carregar ou obter por regra explícita um `TypeRef`. Informação conhecida não pode ser apagada por uma anotação desconhecida. Isso não impõe formato físico nem duplicação de informação derivável.

Nas assinaturas deste conjunto, `T: Type` sempre designa domínio conhecido; `R: TypeRef` admite ambas as formas. `Expression<T>` e `Place<T>` abreviam `Expression<known(T)>` e `Place<known(T)>`. Essa abreviação é apenas notacional, nunca conversão.

### 1.2 Precondições de domínio

Uma exigência de domínio `T` só é satisfeita por `known(T)`. Exigir o mesmo tipo significa exigir o mesmo domínio conhecido, inclusive identificador, versão e parâmetros do tipo de extensão. `unknown_type` NÃO satisfaz essa exigência, mesmo quando os dois operandos compartilham a mesma lacuna. Não há coerção, compatibilidade universal ou inferência de tipo a partir do operador pretendido.

Assim, desconhecimento do valor com `known(int)` permite `add`; desconhecimento do tipo não permite `add`, `concat`, `not`, comparações, conversões ou outros operadores que exigem domínio conhecido. `eq` e `ne` sobre um tipo de extensão exigem também que a igualdade esteja definida por seu contrato. Identificar `opaque_type` não fornece essa operação implicitamente.

Sem conhecer o domínio, continuam admissíveis `read`, dependências de `unknown`, operandos de `opaque` e efeitos de `havoc`, respeitadas suas demais precondições. Nenhuma dessas formas interpreta o valor como inteiro, texto ou booleano. Uma construção observada que não satisfaça uma assinatura precisa DEVE conservar operandos, efeitos e incertezas por abstração apropriada; publicar a operação precisa com precondição violada é IR inválida.

## 2. Literais

Um literal tem domínio conhecido `T`, valor nesse domínio e `TypeRef=known(T)`. Não existe literal com `unknown_type`. Sua grafia-fonte pertence à origem. Strings vazias, zeros, sinais e bytes `00` são valores válidos e NÃO DEVEM ser confundidos com ausência. Literais de `decimal` podem ter várias grafias para o mesmo valor; comparação não depende da grafia.

Um literal de recurso é um nome no namespace declarado, não prova de carregamento, existência ou execução do recurso.

## 3. Expressões

Expressões do núcleo são puras: não escrevem memória, não invocam código, não transferem controle e não lançam exceções. Podem ler estado através de `read`. São avaliadas no ponto de uso, no estado anterior à operação. Compartilhar sua representação NÃO as transforma em valores previamente capturados.

| Forma | Assinatura e semântica |
| --- | --- |
| `literal(T,v)` | Valor `v` do tipo `T` |
| `read(place)` | Valor atual do local designado; conserva o `TypeRef` do local |
| `unknown(R, dependencies, remainingReads, reason)` | Valor não determinado, com `R: TypeRef`, dependências e lacuna de valor explícitas |
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

Os parâmetros `s`, `n` e `pad` são valores constantes nesta V2. `slice_text` só é admissível como expressão pura se a validade do intervalo estiver assegurada por fatos ou pelo domínio de entrada explicitado. Caso contrário, o produtor DEVE materializar teste/saída excepcional ou operação opaca com envelope apropriado; não pode inventar clipping ou ignorar erro possível.

As operações numéricas não têm conversões implícitas. `half_even` escolhe o valor mais próximo e, no empate, o coeficiente inteiro par na escala de destino. `toward_zero` descarta a fração excedente em direção a zero. Divisão geral, truncamento de precisão, representação binária e ordenação de texto não são operações exatas do núcleo.

## 4. Desconhecimento não é constante

`unknown(R,...)` não é uma constante com uma grafia especial. Avaliações distintas não têm igualdade ou correlação garantida. A identidade de uma expressão desconhecida não autoriza concluir `eq(unknown,unknown)=true`.

`unknown(known(T),...)` desconhece o valor dentro do domínio conhecido `T`. `unknown(unknown_type(u),...)` desconhece também o domínio: a lacuna `u` de tipo e a razão do valor DEVEM permanecer distinguíveis. A razão de valor não substitui a referência de tipo nem vice-versa. A forma abreviada `unknown(T,...)` significa apenas `unknown(known(T),...)` quando `T: Type`.

As dependências conhecidas de um valor desconhecido DEVEM ser preservadas. `remainingReads=none` afirma que a lista de leituras é completa; `remainingReads=scope` admite outras leituras dentro do escopo. Isso não permite escritas ou controle oculto. Uma construção cujo efeito pode ser impuro DEVE ser uma operação, não `unknown` dentro de uma expressão.

Cada dependência conhecida conserva sua ocorrência de leitura e seu próprio `TypeRef`, que pode ser desconhecido e não precisa coincidir com o do resultado. `unknown(known(bool), dependencies=[read(p)],...)` só afirma resultado booleano se esse domínio e a pureza forem estabelecidos independentemente; não converte `read(p)` em booleano.

Um predicado puro de valor desconhecido e `TypeRef=known(bool)` pode alimentar `branch`: os dois destinos continuam visíveis. Um predicado de tipo desconhecido não satisfaz essa assinatura. Falta de semântica do predicado não equivale, por si só, a desconhecimento da estrutura de controle.

## 5. Referências e locais

Um `Place` descreve onde um valor é lido ou escrito. Formas V2:

```text
object(ObjectId)
choice(candidates: Place[], remainder: none | MemoryScope, typeRef: TypeRef)
region_slice(RegionId, offset, length, codec)          [memory.regions@1]
```

`object` designa uma entidade já identificada, não inicia lookup por nome. `choice` preserva alternativas ainda não selecionáveis; uma ocorrência não pode escolher a primeira. `remainder=none` declara alternativas exaustivas. Lista vazia com `remainder=none` não designa local algum e é inválida para `read` ou `assign`.

`object(id)` obtém seu `TypeRef` do objeto; uma vista o obtém de seu domínio lógico e contrato de codec, conforme [03](03-memoria-e-aliases.md). `read(place)` conserva esse conhecimento, `ObjectId`/`StorageId` e alternativas conhecidos, identidade da ocorrência, leituras de endereço, proveniência do uso e da declaração e lacunas aplicáveis. Tipo desconhecido, por si só, não apaga a leitura nem abre um armazenamento já conhecido. As condições de validade de endereço, limites e pureza continuam obrigatórias.

O `typeRef` de `choice` é `known(T)` somente quando todas as alternativas, inclusive todo restante possível, asseguram esse mesmo domínio. Se não houver um domínio comum estabelecido, usa `unknown_type(u)` e preserva os `TypeRef` próprios dos candidatos, inclusive domínios conhecidos distintos. Uma escolha fechada de candidatos todos `known(T)` conserva `known(T)`. A forma desconhecida não é um tipo união nem autoriza `assign` ou operadores de domínio conhecido. Leituras conservam as alternativas sem selecionar uma pela ordem. Uma incerteza nominal não deve ser normalizada para certeza apenas porque todas as alternativas têm a mesma grafia.

`MemoryScope` é uma união tipada de objetos, regiões, células, armazenamento visível à unidade ou todo o armazenamento da publicação/ambiente. Toda referência aberta DEVE incluir o restante externo potencial quando ele não puder ser excluído. Escopo textual como “outros dados” sem significado de conjunto é insuficiente.

## 6. Papéis e identidade de operandos

Cada ocorrência de operando possui identidade, posição, `TypeRef` quando denota valor/local e papel. Seu `TypeRef` é o da expressão ou local designado, não uma anotação que possa fortalecê-lo para satisfazer o papel. Referências estruturais, como `LabelId` direto, conservam seu domínio de identidade e não recebem tipo de valor fictício. Os papéis padrão são `VALUE_READ`, `VALUE_WRITE`, `ADDRESS_READ`, `PREDICATE`, `CALL_TARGET`, `ARGUMENT_VALUE`, `ARGUMENT_REFERENCE`, `RESULT_TARGET`, `RESOURCE_TARGET` e `CONTROL_TARGET`.

Papel não substitui semântica da operação. Um argumento por referência não prova que o conteúdo é lido ou escrito. Um alvo de chamada calculado é uma leitura de valor mesmo que seu papel seja `CALL_TARGET`. Calcular o índice de um destino também é leitura; a atualização do destino não apaga essa leitura anterior.

Uma expressão composta permite derivar recursivamente suas leituras. O mesmo objeto lido duas vezes pode ter duas ocorrências. Os consumidores podem agregar efeitos, mas devem manter correlação suficiente para explicar cada uso.

## 7. Captura e ordem

Em `assign(dst, value)`, o endereço de `dst` e `value` são determinados no mesmo estado anterior; a escrita ocorre depois. Uma atribuição de valor NÃO equivale a guardar uma expressão para reavaliar no futuro.

Quando uma construção-fonte avalia uma expressão uma única vez e reutiliza o resultado após atualizações, o produtor DEVE capturá-lo em objeto auxiliar próprio ou usar extensão com semântica equivalente. Esse objeto tem identidade e origem derivada, não é um símbolo-fonte inventado.

Operadores booleanos puros são estritos. Short-circuit com operações impuras DEVE ser explicitado por controle. Não é permitido esconder chamadas em operandos de `and` ou `or`.
