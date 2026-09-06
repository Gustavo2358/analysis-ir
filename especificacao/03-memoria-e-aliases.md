# 03 — Armazenamento, regiões e aliases

**Analysis IR 2.0.0 — Normativo**

## 1. Separação de identidades

`ObjectId` identifica uma declaração ou objeto lógico. `StorageId` identifica armazenamento declarado. Uma análise de storage associa ocorrências de locais a instâncias e intervalos. Esses três conceitos NÃO DEVEM ser unificados por conveniência.

A IR transporta fatos e restrições declarativas de armazenamento; o produto Storage Semantics calcula suas consequências para aliases e acessos. Nenhum consumidor pode presumir que objetos com nomes ou IDs distintos são disjuntos.

## 2. Duração e visibilidade

Cada armazenamento conhecido tem duração `activation`, `persistent` ou `external` e visibilidade `private`, `shared` ou `unknown`.

`activation` cria uma instância por ativação da unidade proprietária. `persistent` conserva estado entre ativações. `external` representa armazenamento cuja existência ou conteúdo é governado pelo ambiente. Uma identidade de célula de ativação é interpretada junto do contexto de invocação; análises podem abstrair contextos, mas devem declarar a perda de precisão.

`private` não significa inalcançável por uma chamada que recebe referência para o local. O escopo de acesso externo inclui referências passadas, capturas, estado compartilhado e armazenamento externo admissível. Sem prova de confinamento, o envelope deve ser ampliado, não reduzido.

## 3. Células abstratas — núcleo

Uma `Cell` contém exatamente um valor de um domínio, cujo conhecimento é declarado por `TypeRef`. `unknown_type(u)` não transforma a célula em armazenamento de tipo dinâmico/universal; apenas deixa seu domínio não estabelecido. Seu tamanho em bytes não é definido. Uma associação `object → cell` permite análise escalar sem exigir representação física. Objeto e célula associados sem conversão compartilham o mesmo `TypeRef`; um alias exato também o conserva. Se o domínio estiver estabelecido nessa associação, todos conservam `known(T)`. Reutilizar uma lacuna entre células distintas não prova compatibilidade de seus domínios.

Identidade de objeto/célula e alias exato da mesma vista sustentam `sameDomain` independentemente de identificar o domínio concreto. Assim, uma cópia de valor entre esses sujeitos não precisa perder sua relação de valor por `TYPE_UNKNOWN`. Sobreposição de bytes, alias possível ou identidade de região com vistas diferentes não fornecem essa prova. Escritas mudam conteúdo, não o domínio estável declarado da célula; nenhuma dessas regras presume codec ou conversão.

Células com identidades distintas representam armazenamento independente **apenas quando essa separação foi estabelecida pelo produtor ou declarada como premissa rastreável**. O produtor NÃO DEVE criar células distintas para contornar um layout ou alias desconhecido. Dois objetos que nomeiam o mesmo armazenamento DEVEM compartilhar a célula, possuir relação de alias explícita ou permanecer abertos.

Uma célula admite atualização completa atômica. Escrita parcial em célula sem decomposição não é precisa: deve ser representada em uma região, transformada em leitura-modificação-escrita com semântica comprovada ou abstraída conservadoramente.

## 4. Associações declarativas

Cada objeto possui uma das seguintes associações:

| Associação | Significado |
| --- | --- |
| `cell(storageId)` | Todo o valor reside na célula identificada |
| `view(regionId, offset, extent, codec)` | Vista de intervalo de bytes, com interpretação explícita |
| `alias(objectId)` | Mesmo local e mesmo tipo/vista do objeto referido |
| `alternatives(bindings, remainder)` | Conjunto de associações possíveis, possivelmente aberto |
| `unknown(scope, reason)` | Associação não determinada dentro do escopo |

`alias` não pode formar ciclo sem um armazenamento-base resolúvel. Uma relação de sobreposição conhecida, mas sem offset conhecido, não deve virar `alias` exato; usa alternativas/restrições abertas. Fatos conhecidos contraditórios de tipo, duração ou limites são inválidos; lacunas reais usam incerteza explícita, nunca seleção silenciosa de uma associação. Desconhecimento de associação não implica desconhecimento de tipo, e `unknown_type` não substitui `unknown(scope, reason)`. Em `alternatives`, o domínio comum segue a regra de `choice`; domínios próprios dos candidatos são preservados.

## 5. Regiões de bytes — `memory.regions@1`

Uma `Region` denota uma sequência de octetos. Sua extensão é inteira não negativa ou desconhecida. Uma vista `view(r,o,n,c)` cobre o intervalo semiaberto `[o,o+n)`; offsets começam em zero. `n=0` representa intervalo vazio e não escreve bytes.

Vistas podem sobrepor-se. Intervalos em regiões-base comprovadamente distintas são disjuntos; vistas sobre a mesma base são comparadas pelos intervalos. Regiões cuja relação não está determinada não podem ser tratadas como distintas por ID nominal.

Offsets e extensões declarativos são constantes na forma precisa básica. `region_slice` permite offset calculado no ponto de uso; seu resultado deve identificar todos os intervalos possíveis e os limites de validade. Um offset desconhecido dentro de uma região finita é acesso potencial a todos os intervalos compatíveis, não apenas ao primeiro.

A unidade de offset físico é sempre **octeto**. Caractere lógico, elemento de array e octeto não são unidades intercambiáveis. Um produtor deve fornecer a conversão relevante; uma análise não a recupera do nome do campo.

## 6. Codecs padronizados da extensão

| Codec | Valor lógico | Regra |
| --- | --- | --- |
| `bytes.identity@1` | `bytes` | Identidade; comprimento deve igualar extensão da vista |
| `text.ascii@1` | `text` | Um caractere U+0000..U+007F por octeto; comprimento deve igualar extensão |
| `unsigned.binary@1(width, order)` | `int` | `width` positivo e múltiplo de 8; `order=little` ou `order=big`; domínio `0..2^width-1` |
| `signed.twos_complement@1(width, order)` | `int` | Mesmas restrições de largura; domínio `-2^(width-1)..2^(width-1)-1` |

Codecs adicionais são extensões versionadas. Não há codificação ambiental padrão. Um codec desconhecido preserva bytes e identidade, mas bloqueia a decodificação exata; não bloqueia automaticamente uma cópia bruta de bytes.

O domínio lógico estabelecido de uma vista é `known(T)`, mesmo se sua codificação não puder ser interpretada. Os codecs padronizados acima estabelecem seus domínios conhecidos. Se o próprio domínio lógico não foi estabelecido, a vista usa `unknown_type(u)` e conserva os fatos físicos conhecidos; não pode alegar um codec que estabeleça domínio contraditório. `region_slice` obtém o mesmo `TypeRef` desse contrato lógico. Uma leitura com interpretação indisponível só conserva a forma pura `read` se totalidade e ausência de efeitos excepcionais estiverem asseguradas; caso contrário, exige abstração por operação com envelope. Uma escrita precisa por codec continua exigindo domínio conhecido e codificação aplicável.

Leitura de sequência inválida para o codec ou escrita fora de seu domínio exige caminho de erro/abstração explícita. O produtor só pode usar operação pura e total sobre um domínio cuja validade esteja assegurada. Texto não representável não deve ser substituído automaticamente por `?`.

`assign` em vista grava a codificação exata do valor e exige tamanho/domínio compatível. Preenchimento e truncamento são operações explícitas. `copy_bytes` transfere a imagem de bytes capturada antes da escrita, inclusive sob sobreposição; não aplica codecs.

## 7. Arrays e seleções

O núcleo não exige uma espécie nominal “array”. Uma seleção pode ser reduzida a uma vista de região com offset explícito quando base, extensão, stride e ajuste de índice forem conhecidos. A avaliação do índice é uma leitura identificável.

Índice ou stride não conhecido torna o acesso conservador sobre o domínio admissível. Possível violação de limites deve preservar o comportamento excepcional ou aberto. Não é permitido assumir índices válidos apenas porque os testes usam índices válidos.

Não se exige expansão de todos os elementos em declarações independentes. Uma representação compacta é permitida se conserva cardinalidade lógica, fronteiras, alvos e incertezas observáveis.

## 8. Inicialização

A mera declaração de objeto NÃO é uma definição executada. Não existe zero inicial implícito. Na entrada de uma ativação, o estado de cada célula/região vem de um `EntryState` explícito: parâmetro, valor inicial conhecido, estado externo, estado persistente anterior ou valor não inicializado/desconhecido.

O consumidor de reaching definitions DEVE identificar definições de entrada separadas das operações. Uma inicialização que ocorre em fluxo ordinário deve ser uma operação, não ser repetida a cada entrada por interpretar um atributo declarativo.

Quando várias entradas têm estados iniciais diferentes, a distinção deve ser preservada. Chamadas locais compartilham a ativação atual e NÃO reinicializam células.

Um `EntryState` é um inventário por entrada de condições iniciais de armazenamento, com origem/premissa. Cada condição associa um local a `literal(value)`, `parameter(position)`, `preserve`, `external_unknown` ou `uninitialized`. `parameter` captura o argumento de valor da ativação; um parâmetro por referência associa o objeto ao local recebido, sem criar cópia independente. `preserve` conserva armazenamento persistente/externo existente, cuja abstração inicial pode ser desconhecida. `uninitialized` indica ausência de valor inicial assegurado, não um literal zero.

Condições não fornecidas são abertas: para células novas, valor não inicializado/desconhecido; para estado persistente/externo, conteúdo preservado não determinado. Condições iniciais sobre aliases ou vistas sobrepostas DEVEM ser consistentes entre si. Dois literais incompatíveis para o mesmo byte/célula na mesma entrada constituem premissas contraditórias e não podem ser conciliados por ordem do inventário.

`external_unknown` e `uninitialized` conservam o `TypeRef` do local; não fazem seu tipo virar desconhecido. Um literal inicial conserva seu domínio conhecido e exige `sameDomain` com o local, além das condições de representação aplicáveis; uma anotação `unknown_type` sozinha não prova compatibilidade. A inicialização precisa por parâmetro obedece às condições de transmissão da assinatura, inclusive prova de mesmo domínio sem identificá-lo concretamente; falta de prova conserva a condição/lacuna sem fabricar um valor ou um domínio. A prova do vínculo declarado em `EntryState` é verificada em `entry_site(e)`, conforme [02, §1.4](02-tipos-valores-e-operandos.md#14-escopo-de-provas-de-domínio); uma premissa limitada à fronteira de uma chamada não valida a inicialização para todas as entradas/ativações.

`EntryState` descreve a entrada da ativação, não uma operação que se repete em cada visita ao label inicial por back-edge. Reentrar no label por transferência dentro da mesma ativação não reaplica seus seeds.

## 9. Atualizações fortes e fracas

Uma atualização forte exige destino exato e sobrescrita completa da localização relevante em todos os comportamentos representados daquela transição. Só nessas condições uma definição anterior pode ser inteiramente eliminada.

Uma atualização fraca acrescenta possibilidade de nova definição sem eliminar anteriores não comprovadamente sobrescritas. Destino ambíguo, alias aberto, índice não determinado ou `may_write` exigem atualização fraca sobre as localizações potencialmente afetadas.

Escrita parcial elimina definições anteriores apenas no intervalo comprovadamente sobrescrito. Uma consulta do objeto inteiro pode depender de várias definições que contribuem com fragmentos; não se escolhe uma definição “vencedora” para todos os bytes.

## 10. Desconhecimento e efeitos

Um efeito desconhecido limitado a uma região não contamina automaticamente armazenamento comprovadamente disjunto. Um efeito cujo escopo não é delimitável abrange todo o armazenamento potencialmente visível, incluindo a parte externa aberta.

Preservar fatos independentes NÃO significa preservar valores obsoletos após sobrescrita obrigatória. `havoc.must` de uma célula exata remove o valor anterior como valor corrente; o evento anterior permanece no inventário histórico. `havoc.may` conserva a alternativa sem escrita e acrescenta o restante desconhecido.
