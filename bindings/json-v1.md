# Analysis IR JSON Binding 1.0.0

**Status: DRAFT — não NORMATIVE / ACCEPTED.** Este PR mantém a candidata em
revisão; seu merge não a promove automaticamente a contrato estável.
**Targets AIR 2.0.0**, na edição em fechamento deste repositório.
**Transport contract != semantic version**: `bindingVersion` identifica transporte;
`airVersion` identifica semântica; nenhuma delas é versão de biblioteca.

A autoridade é a [especificação AIR](../especificacao/00-escopo-e-convencoes.md).
Este documento codifica seus fatos; não acrescenta precondições, entidades ou
resultados de análise. Divergência é defeito do binding. As decisões desta revisão
estão no [handoff](revisao-json-v1.md); o estado da edição e o impacto normativo
estão na [política de versões](../especificacao/09-extensibilidade-e-compatibilidade.md#52-fechamento-do-modelo-antes-da-estabilização-da-200).

## 1. Correspondência e alcance

```text
modelo semântico AIR ⇄ codificação JSON
```

Um codec pode ser implementado em qualquer linguagem lendo este documento e os
normativos referidos. Classes, factories, enums de runtime e estruturas privadas
de uma implementação não são fontes de campos. Não se serializam callbacks,
ponteiros, nomes de classes, identidade de objetos em memória, CFG, efeitos
calculados, RD ou valores propagados.

Esta candidata cobre o núcleo e as extensões padronizadas `memory.regions@1`,
`control.local@1` e `control.indirect@1`. Identidades de outras capacidades podem
ser transportadas; seu payload preciso requer binding de extensão negociado.
Não há payload semântico livre nem ativação de código por nome de classe. Sem
codificação precisa disponível, uma redução/abstração AIR já publicada conserva
operandos, envelopes e incertezas; o decoder não inventa essa redução.

A promoção exige revisão explícita e evidência dos oráculos da seção 13.
Este repositório não contém codec nem anuncia testes de round-trip executados.
A candidata anterior baseada no catálogo Java é incompatível com esta revisão;
não recebe promessa de leitura ou conversão silenciosa.

## 2. Envelope e versão

```json
{
  "binding": "analysis-ir-json",
  "bindingVersion": "1.0.0",
  "airVersion": "2.0.0",
  "publication": {
    "id": {"domain": "publication", "localId": "example-publication"},
    "capabilities": {"required": [], "provided": []},
    "artifacts": [],
    "units": [],
    "storage": [],
    "resources": [],
    "artifactRelations": [],
    "origins": [],
    "coverage": {
      "inventory": "COMPLETE",
      "scope": {
        "kind": "publication",
        "publication": {"domain": "publication", "localId": "example-publication"}
      },
      "items": [],
      "uncertainties": []
    },
    "uncertainties": [],
    "premises": []
  }
}
```

O exemplo representa zero conhecido no escopo declarado; não representa input
indisponível. Os inventários e campos do envelope são obrigatórios.
`airVersion` codifica, uma única vez, o componente semântico `Publication.version`;
`publication.id` codifica `publicationId`. Não existe campo `contracts`.

Esta candidata aceita exatamente `bindingVersion="1.0.0"` e `airVersion="2.0.0"`.
Outra combinação exige negociação/revisão explícita. Semelhança física não prova
compatibilidade. Após estabilização do binding, remover/alterar formas ou defaults
incompativelmente exige major de transporte; isso não altera por si só a versão AIR.

## 3. Convenções físicas e números

O documento é exatamente um objeto JSON em UTF-8, sem BOM. Só whitespace JSON é
permitido depois dele. Rejeitam-se propriedades duplicadas, UTF-8 inválido e
surrogates isolados, inclusive em escapes. Strings denotam escalares Unicode;
não se normalizam caixa, padding ou Unicode.

As tabelas usam esta metanotação, independente de linguagem:

| Notação | Codificação |
| --- | --- |
| `Text` | String de escalares Unicode |
| `Bool` | `true` ou `false` JSON |
| `Integer` | String decimal canônica, padrão `0` ou `-?[1-9][0-9]*` |
| `Natural` | `Integer` não negativo, sem limite semântico fixo |
| `X[]` | Array de X; ordem conforme seção 6 |
| `X?` | Campo presente com X ou `null`, apenas para ausência permitida |
| `f:X` | Propriedade obrigatória de nome `f` e tipo X |
| `v(f:X,...)` | Objeto com `kind:"v"` e os campos indicados |

**Todos os inteiros**, inclusive posições, escalas, larguras, contagens e coordenadas,
usam strings. Não há limite de `int` Java, IEEE-754 ou 2^53 no contrato. Não usar
`-0`, `+1`, zeros à esquerda, fração ou expoente. Limite de implementação é erro
operacional explícito, nunca truncamento ou nova regra de validade AIR.

| Valor literal | Forma JSON |
| --- | --- |
| bool | `bool(value:Bool)` |
| int | `int(value:Integer)` |
| decimal | `decimal(coefficient:Integer,scale:Natural)` |
| text | `text(value:Text)` |
| bytes | `bytes(base64:Text)` |
| label | `label(label:LabelId,domain:LabelType)` |

`bytes.base64` usa alfabeto padrão `A-Z a-z 0-9 + /`, grupos de quatro caracteres,
padding `=` necessário, bits de padding zero e nenhum whitespace; vazio codifica
zero octetos. Não se publica array de inteiros como outra forma de bytes.
`decimal` conserva coeficiente e escala; por exemplo
`{"kind":"decimal","coefficient":"125","scale":"2"}`. Igualdade numérica
em `dispatch` independe de grafias decimais distintas. Não existe literal com tipo
desconhecido. Literal de extensão exige seu binding próprio.

## 4. Identidades e fechamento

Toda ocorrência de ID, inclusive referência, transporta domínio e namespace completo:

```text
PublicationId = {domain:"publication",localId:Text}
GlobalId = {domain:Text,publication:Text,localId:Text}
UnitOwnedId = {domain:Text,publication:Text,unit:Text,localId:Text}
OperandId = {domain:"operand",publication:Text,unit:Text,
             owner:{kind:"operation"|"entry",localId:Text},localId:Text}
```

`publication` e `unit` contêm os `localId` dos respectivos proprietários. `localId`
é opaco. Esses objetos não autorizam usar nomes de exibição como joins.

| Forma | Domínios / entidades AIR |
| --- | --- |
| GlobalId | `artifact`, `relation` (`ArtifactRelationId`), `unit`, `storage`, `resource`, `origin`, `uncertainty`, `premise` |
| UnitOwnedId | `entry`, `label`, `operation`, `object`, `completion_port` |
| OperandId | Ocorrência de expressão/local na operação ou condição de entrada |

`RegionId` e identidade de célula são `StorageId` com espécie verificada. Não se
adicionam domínios `region`, `cell` ou `contract`. `ContractRef` é valor, não ID.
A unidade proprietária de armazenamento de ativação é propriedade declarativa;
seu ID permanece global, conforme [01](../especificacao/01-modelo-e-identidades.md).

Há uma definição por ID completo. Repetir um ID como referência não define outro
fato. Cada nó de expressão/local é definido uma vez sob seu proprietário. Envelopes,
recursos e resultados opacos referem essas ocorrências por ID quando indicado;
não duplicam a árvore nem criam uma segunda avaliação. Usos executáveis distintos
exigem ocorrências distintas, mesmo se a expressão tem a mesma grafia.

## 5. Variantes, ausência e desconhecimento

Uma soma usa `kind` obrigatório; não se escolhe variante pela presença de campos.
Todos os campos das tabelas são obrigatórios. Apenas `?` admite `null` para ausência
real. Campo omitido não vira `null`, zero, `false`, `[]` ou `none`. Uma lacuna não
é ausência e usa sua variante/referência explícita.

```json
{"kind":"known","type":{"kind":"text"}}
```

```json
{"kind":"unknown_type","uncertainty":{"domain":"uncertainty","publication":"p","localId":"type-1"}}
```

| Tipo de transporte | Variantes |
| --- | --- |
| TypeRef | `known(type:Type)`; `unknown_type(uncertainty:UncertaintyId)` |
| Type | `bool`; `int`; `decimal`; `text`; `bytes`; `opaque_type(name:Text,version:Text)`; LabelType |
| LabelType | `label(unit:UnitId,labels:LabelId[])` |
| UnknownBound | `none`; `unknown(uncertainty:UncertaintyId)` |
| ContractKnowledge | `known(reference:ContractRef)`; `unknown(uncertainty:UncertaintyId)` |
| ContractRef | `{authority:Text,version:Text,evidence:OriginId[]}` |

`TypeRef.unknown_type` fecha sobre `TYPE_UNKNOWN`; contrato desconhecido fecha sobre
`CONTRACT_UNKNOWN`. Evidência de `ContractRef` é não vazia, conforme 01, §9. Sua versão
é a revisão da autoridade, sem exigir semver de biblioteca. Tipos/capacidades de
extensão conservam sua versão declarada, com compatibilidade regida pelo manifesto.

Tipo, valor, aridade, modo, associação, interpretação de codec e suporte a extensão
continuam conhecimentos distintos. Razão de valor não substitui razão de tipo.
Nenhum `known` do binding tem significado mais forte que seu fato AIR.

## 6. Ordenação e canonical JSON

A forma canônica tem propriedades em ordem lexicográfica por valores escalares
Unicode, sem indentação e sem newline final. Escapa apenas `"`, `\` e U+0000..U+001F,
usando `\b`, `\f`, `\n`, `\r`, `\t` onde cabíveis e `\u00xx` minúsculo nos demais.
Não escapa `/` nem outros escalares; a saída é UTF-8.

Arrays preservam sua ordem transportada. Operações, argumentos, posições de assinatura,
resultados e cadeias de inclusão têm a ordem semântica pertinente. Inventários,
sequências, conjuntos de premissas, condições de entrada e casos de `dispatch`
preservam uma ordem física para round-trip; essa ordem não introduz execução,
precedência de condições sobrepostas nem prioridade entre casos. Duplicatas são
rejeitadas quando violam unicidade de identidades, posições ou conjuntos do modelo.

Para a mesma publicação com a mesma ordem física, a escrita canônica produz os
mesmos bytes. Não se exige identidade de bytes entre decomposições equivalentes,
renomeações ou permutações de inventários. O encoder não acrescenta timestamp,
IDs aleatórios ou propriedades do processo. Preserva metadados e IDs já publicados
quando admitidos pelo catálogo; não exige que `publicationId` seja hash do JSON.

## 7. Operações e ocorrências

```text
Sequence = {label:LabelId,instructions:Operation[],terminator:Operation,origin:OriginId}
OperationHeader = {id:OperationId,origin:OriginId,coverage:CoverageStatus,
                   precision:Precision,uncertainties:UncertaintyId[]}
OperandHeader = {id:OperandId,role:OperandRole,origin:OriginId}
```

`instructions` contém apenas operações comuns; `terminator` contém exatamente um
terminador. As espécies core são versionadas por `airVersion`; operações das
extensões padronizadas também exigem a capacidade correspondente. Não se publicam
nomes Java como espécies. A sequência proprietária vem da posição declarativa;
a unidade vem do ID e ambas são verificadas por fechamento/unicidade.

Todas as formas abaixo têm `header:OperationHeader`, além de `kind` e seus campos:

| kind | Campos | Classe |
| --- | --- | --- |
| assign | `destination:Place,value:Expression` | comum |
| havoc.must | `destination:Place,reason:UncertaintyId` | comum |
| havoc.may | `scope:MemoryScope,reason:UncertaintyId` | comum |
| nop | nenhum | comum |
| copy_bytes | `destination:ByteRange,source:ByteRange,length:Natural,fallback:Envelope` | comum, memory.regions@1 |
| jump | `destination:LabelId` | terminador |
| branch | `predicate:Expression,trueDestination:LabelId,falseDestination:LabelId` | terminador |
| dispatch | `selector:Expression,cases:Case[],defaultDestination:LabelId` | terminador |
| invoke | `action:Text,target:Target,arguments:Argument[],results:Place[],signature:InvocationSignature,effectOperands:Place[],effectBound:EffectBound,outcomes:InvocationOutcomes,contract:ContractKnowledge` | terminador |
| return | `values:Expression[]` | terminador |
| raise | `tag:Text,values:Expression[]` | terminador |
| halt | `haltKind:HaltKind` | terminador |
| opaque | `observedKind:Text,knownOperands:Operand[],valueResults:OperandId[],envelope:Envelope` | terminador |
| local.invoke | `entry:LabelId,completionPorts:CompletionPortId[],resume:LabelId,fallback:Envelope` | terminador, control.local@1 |
| local.boundary | `port:CompletionPortId,defaultDestination:LabelId,fallback:Envelope` | terminador, control.local@1 |
| local.resume | `fallback:Envelope` | terminador, control.local@1 |
| local.unwind | `count:Natural,destination:LabelId,fallback:Envelope` | terminador, control.local@1 |
| indirect.jump | `target:Expression,within:LabelType,fallback:Envelope` | terminador, control.indirect@1 |

`Case={value:LiteralValue,destination:LabelId}`. `ByteRange` é definido na seção 10.
`Operand` é Expression ou Place, cujos discriminadores são distintos. O tipo/posição
vem da forma e do proprietário, sem anotação que fortaleça um `TypeRef`.

`effectOperands` é o local físico para definir os locais contratuais de 04, §7.4
usados apenas pelo limite. Não adiciona leitura/escrita pela mera presença. Locais
já definidos nos argumentos/resultados são referidos por ID no limite. Os IDs de
`opaque.valueResults` referem Places definidos em `knownOperands`, sem nova avaliação.
`opaque` codifica `reasons` em `header.uncertainties`, sem segunda lista divergente.
Não há `signatureGaps` paralelo aos restantes/lacunas da assinatura nem `return.entryScope`.

## 8. Expressões e locais

Cada forma tem `header:OperandHeader`, além dos campos abaixo. Seu `TypeRef` é obtido
pelas regras de [02](../especificacao/02-tipos-valores-e-operandos.md); o codec não
infere conhecimento de tipo pela operação pretendida.

| Soma | kind e campos |
| --- | --- |
| Expression | `literal(value:LiteralValue)`; `read(place:Place)`; `unknown(typeRef:TypeRef,dependencies:Expression[],remainingReads:MemoryBound,reason:UncertaintyId)` |
| Expression composta | `unary(operator:UnaryOperator,argument:Expression)`; `binary(operator:BinaryOperator,left:Expression,right:Expression)`; `quantize(value:Expression,scale:Natural,rounding:Rounding)`; `fit_text(value:Expression,length:Natural,pad:Text)`; `slice_text(value:Expression,start:Expression,count:Expression)`; `trim_right(value:Expression,characters:Text)` |
| Place | `object(object:ObjectId)`; `choice(candidates:Place[],remainder:MemoryBound,typeRef:TypeRef)`; `region_slice(region:StorageId,offset:Expression,length:Expression,codec:Codec,typeRef:TypeRef)` |

`pad` contém exatamente um escalar. Operandos de índices/comprimentos calculados
exigem `known(int)`. `region_slice` exige `memory.regions@1`; seu `typeRef` deve
coincidir com o conhecimento lógico do codec/vista. Não há `boundsProof`, `accessProof`
ou `knownRemainderDomainProof`. A ausência desses campos não dispensa precondições
nem a garantia universal sobre candidatos/restante de uma escolha (02 e 06, §5.1).

## 9. Interações, assinaturas e premissas

| Estrutura | Forma |
| --- | --- |
| Target | `internal(entry:EntryId)`; `literal(category:Text,namespace:Text,name:Text,namePolicy:NamePolicy,origin:OriginId)`; `computed(category:Text,namespace:Text,name:Expression,namePolicy:NamePolicy,origin:OriginId)` |
| NamePolicy | `exact`; `extension(name:Text,version:Text)`; `unknown(uncertainty:UncertaintyId)` |
| Argument | `value(value:Expression)`; `copy(value:Expression)`; `reference(place:Place)` |
| InvocationSignature | `entry(entry:EntryId)`; `external(signature:Signature)` |
| Signature | `{parameters:ParameterInventory,results:ResultInventory,origin:OriginId}` |
| ParameterInventory | `{known:Parameter[],remainder:UnknownBound}` |
| ResultInventory | `{known:ResultSlot[],remainder:UnknownBound}` |
| Parameter | `{position:Natural,mode:ModeKnowledge,typeRef:TypeRef,objectBinding:ParameterBinding,origin:OriginId}` |
| ResultSlot | `{position:Natural,typeRef:TypeRef,origin:OriginId}` |
| ModeKnowledge | `known(mode:PassingMode)`; `unknown(uncertainty:UncertaintyId)` |
| ParameterBinding | `object(object:ObjectId)`; `unknown(uncertainty:UncertaintyId)`; `external` |
| Premise | `{id:PremiseId,authority:Text,justification:Text,origin:OriginId,assertion:Assertion}` |
| Assertion | `same_domain(left:DomainSubject,right:DomainSubject,scope:DomainProofScope)`; `disjoint_storage(storage:StorageId[])` |

Posições são codificadas a partir de `"0"`; inventário fechado é contíguo, ordenado
e completo. Em inventário aberto, `known` preserva cada posição conhecida, inclusive
modo/tipo desconhecidos, e o restante admite outras posições não enumeradas.
`Parameter.objectBinding` conserva objeto da entrada ou sua lacuna própria.
Em assinatura externa usa `external`, pois esse vínculo não se aplica; não usa
`unknown` ou objeto fictício para esse caso. `unknown_type` não abre aridade.

A assinatura interna referencia exatamente a entrada do target. A externa pertence
à operação que a contém. `ContractRef` não aponta para conteúdo externo: assinatura,
efeitos, outcomes, origens e premissas são o conteúdo disponível de 04, §7.4.
Não existe `resource(ResourceId)` em Target. `NamePolicy.extension` e codecs de
extensão referem capacidades com manifesto; não um contrato de chamada sem regra
para nomes/codecs. Sem suporte, não recebem interpretação precisa implícita.

| DomainSubject | Significado normativo em 02, §1.4 |
| --- | --- |
| `object(object:ObjectId)`; `cell(cell:StorageId)`; `operand(operand:OperandId)` | Domínio de declaração/local/expressão |
| `parameter(entry:EntryId,position:Natural)`; `result(entry:EntryId,position:Natural)` | Posição declarada da entrada |
| `call_parameter(invocation:OperationId,entry:EntryId,position:Natural)`; `call_result(invocation:OperationId,entry:EntryId,position:Natural)` | Posição interna instanciada no site |
| `external_parameter(invocation:OperationId,position:Natural)`; `external_result(invocation:OperationId,position:Natural)` | Posição externa materializada no próprio invoke |

`DomainProofScope` usa `publication`, `unit(unit:UnitId)`, `entry(entry:EntryId)`,
`operation(operation:OperationId)`, `invocation(invocation:OperationId)` ou
`intersection(left:DomainProofScope,right:DomainProofScope)` finita. As formas e
sua interseção são exatamente as de 02, §1.4; não há `activation`. Evidência de uma
chamada não se aplica a outra pela igualdade de autoridade/versão.

`disjoint_storage` codifica somente a garantia de 03, §3.1: pelo menos duas bases
distintas, disjunção par a par universal na publicação, sem campo `scope`. Não é
alias inferido, separação de candidatos abertos nem certificado de validação.
`same_domain` conserva lacunas e cobre a escolha inteira quando esse for o sujeito.
Nenhuma dessas formas é `safety`; novas asserções exigem extensão normativa.

## 10. Catálogo dos demais fatos

As tabelas abaixo são definições de transporte. Cada significado vem do documento
normativo indicado; tipos auxiliares são apenas agrupamentos físicos dos seus fatos.

### 10.1 Publicação, unidades, memória e entrada — AIR 01/03

| Estrutura | Campos ou variantes |
| --- | --- |
| Publication | `id,capabilities,artifacts,units,storage,resources,artifactRelations,origins,coverage,uncertainties,premises`, como seção 2, com tipos dos inventários abaixo |
| Capability | `{name:Text,version:Text}` |
| Manifest | `{required:Capability[],provided:Capability[]}` |
| Artifact | `{id:ArtifactId,logicalName:Text,contentDigest:Text?}` |
| Unit | `{id:UnitId,containingUnit:UnitId?,objects:Object[],visibleObjects:ObjectId[],entries:Entry[],sequences:Sequence[],completionPorts:CompletionPort[],body:BodyKnowledge,coverage:Coverage,origin:OriginId}` |
| BodyKnowledge | `available`; `unavailable(uncertainty:UncertaintyId)` |
| CompletionPort | `{id:CompletionPortId,origin:OriginId}` |
| Entry | `{id:EntryId,initialLabel:LabelId?,signature:Signature,state:EntryState,origin:OriginId}` |
| Object | `{id:ObjectId,displayName:Text?,typeRef:TypeRef,storage:StorageBinding,visibility:Visibility,origin:OriginId,coverage:CoverageStatus,precision:Precision}` |
| StorageHeader | `{id:StorageId,owner:UnitId?,lifetime:Lifetime,visibility:Visibility,origin:OriginId}` |
| Storage | `cell(header:StorageHeader,typeRef:TypeRef)`; `region(header:StorageHeader,extent:ExtentKnowledge)` |
| ExtentKnowledge | `known(value:Natural)`; `unknown(uncertainty:UncertaintyId)` |
| StorageBinding | `cell(storage:StorageId)`; `view(region:StorageId,offset:Natural,extent:Natural,codec:Codec)`; `alias(object:ObjectId)`; `alternatives(alternatives:StorageBinding[],remainder:MemoryBound)`; `unknown(scope:MemoryScope,reason:UncertaintyId)` |
| Codec | `bytes.identity`; `text.ascii`; `unsigned.binary(width:Natural,order:ByteOrder)`; `signed.twos_complement(width:Natural,order:ByteOrder)`; `extension(name:Text,version:Text,logicalType:TypeRef)`; `unknown(logicalType:TypeRef,reason:UncertaintyId)` |
| ByteRange | `{region:StorageId,offset:Expression,extent:Expression}` |
| EntryState | `{conditions:InitialCondition[],uncertainties:UncertaintyId[]}` |
| InitialCondition | `{place:Place,value:InitialValue,origin:OriginId,premises:PremiseId[]}` |
| InitialValue | `literal(value:LiteralExpression)`; `parameter(position:Natural)`; `preserve`; `external_unknown(reason:UncertaintyId)`; `uninitialized(reason:UncertaintyId)` |
| Resource | `{id:ResourceId,description:ResourceDescription,origin:OriginId}` |
| ResourceDescription | Target interno/literal; para calculado, os mesmos campos de Target calculado com `name:OperandId` em vez de Expression |
| ArtifactRelation | `{id:ArtifactRelationId,source:ArtifactId,destination:RelationTarget,relationKind:Text,origin:OriginId,coverage:CoverageStatus}` |
| RelationTarget | `internal(artifact:ArtifactId)`; `external(resource:LiteralTarget)` |

`Capability.version` conserva a versão declarada pelo manifesto: nas capacidades
padronizadas, `"1"` codifica `@1`; nos perfis desta edição, `"2"` codifica `@2`.
Não se extrai automaticamente só o major de uma versão de extensão arbitrária.

`LiteralValue` é uma das formas de valor da seção 3. `LiteralExpression` é
Expression de `kind:"literal"`; `LiteralTarget` é Target de `kind:"literal"`. O literal e Place de condição inicial pertencem à entrada. A origem
contratual da condição já pode evidenciar um seed; não exige uma premissa `safety`.

`initialLabel=null` é permitido apenas na declaração de entrada cujo corpo está
explicitamente indisponível. Label presente sempre fecha sobre sequência da unidade.
Corpo disponível exige entradas/sequências. Armazenamento `ACTIVATION` exige unidade
proprietária. Ausência de proprietário em storage externo não é identidade desconhecida.

Os quatro codecs padronizados são versão `@1`, coberta por `memory.regions@1`;
não usam parâmetros implícitos de ambiente. Codecs binários exigem largura positiva
múltipla de oito. `ByteRange.extent` delimita o intervalo; `copy_bytes.length` não
pode excedê-lo. Índices/comprimentos calculados conservam ocorrências e tipos.

`ResourceDescription` calculada referencia a ocorrência de nome já definida numa
operação e seu ponto de avaliação; a declaração não a executa novamente nem captura
seu valor. A descrição não cria uma variante de target por ResourceId. Uma relação
estrutural usa nome literal, sem estado dinâmico necessário para interpretá-la.

### 10.2 Envelopes e pontos — AIR 04/05/06

| Estrutura | Campos ou variantes |
| --- | --- |
| Envelope | `{memory:MemoryEnvelope,control:ControlEnvelope,dependencies:DependencyEnvelope}` |
| MemoryEnvelope | `{knownReads:OperandId[],otherReads:MemoryBound,knownWrites:OperandId[],otherWrites:MemoryBound,mustOverwrite:OperandId[]}` |
| ForeignEffects | `{reads:MemoryBound,writes:MemoryBound,mustOverwrite:OperandId[]}` |
| EffectBound | `{otherwise:ForeignEffects,perOutcome:OutcomeEffects[]}` |
| OutcomeEffects | `{outcome:OutcomeKey,effects:ForeignEffects}` |
| InvocationOutcomes | `{known:InvocationAlternative[],remainder:ControlBound}` |
| InvocationAlternative | `normal(label:LabelId)`; `exception(tag:Text,destination:ExceptionDestination)`; `any_exception(destination:ExceptionDestination)`; `halt`; `diverge` |
| ControlEnvelope | `{known:ControlAlternative[],remainder:ControlBound}` |
| ControlAlternative | InvocationAlternative ou `jump(label:LabelId)`; `return`; `continue` |
| ExceptionDestination | `handler(label:LabelId)`; `propagate` |
| DependencyEnvelope | `{known:ResourceUse[],remainder:DependencyBound}` |
| ResourceUse | `{action:Text,target:ResourceDescription,point:ProgramPoint,origin:OriginId}` |
| ProgramPoint | `before(operation:OperationId)`; `after(operation:OperationId,outcome:OutcomeKey)`; `entry(entry:EntryId)`; `exit(unit:UnitId,outcome:OutcomeKey)` |
| OutcomeKey | `normal`; `exception(tag:Text)`; `other_exception`; `halt`; `diverge` |
| MemoryBound | `none`; `within(scope:MemoryScope)` |
| MemoryScope | `objects(objects:ObjectId[])`; `storage(storage:StorageId[])`; `visible(unit:UnitId,includingExternal:Bool)`; `all(publication:PublicationId,includingEnvironment:Bool)`; `union(members:MemoryScope[])` |
| ControlBound | `none`; `within(scope:ControlScope)` |
| ControlScope | `labels(labels:LabelId[])`; `unit(unit:UnitId,labels:Bool,normalExit:Bool,exceptionalExit:Bool,halt:Bool,diverge:Bool,externalControl:Bool)`; `all(publication:PublicationId)`; `union(members:ControlScope[])` |
| DependencyBound | `none`; `any_resource`; `categories(categories:Text[])` |

As flags de `ControlScope.unit` incluem os conjuntos indicados em 05, §6; `labels`
inclui todos os labels da unidade, inclusive os de entrada. `ControlScope.all`
codifica `any_control`, inclusive ambiente. Excluir parte externa de memória
exige a garantia normativa de confinamento; não é default da codificação.

`InvocationOutcomes` tem no máximo um normal, tags únicas e no máximo um catch-all;
`ControlEnvelope` pode ter vários destinos. `continue` só é permitido no fallback
de operação comum, como `copy_bytes`; não é fallthrough de terminador. `return`
é saída da unidade. `normal(label)` não codifica essa saída. Restante aberto não
é um outcome finito chamado `unknown` e não é eliminado por redundância.

Chaves de `perOutcome` são únicas e correspondem aos resultados admitidos; a regra
`otherwise` cobre os demais, inclusive fronteira aberta. `other_exception` é o
restante de tags, sem competir com tag específica. Menção a outcome/ponto não é
prova de alcançabilidade; divergência não fabrica um próximo estado.

Em uso calculado de `DependencyEnvelope`, o operando de nome pertence à operação
do envelope e o ponto é `before` dessa operação. Referir ocorrência de outra
operação não simula captura; a declaração global de recurso apenas descreve o uso
em seu ponto original.

Referências de leitura/escrita no envelope fecham sobre operandos da operação;
escritas e sobrescritas designam Places. Leitura conhecida pode referir uma leitura
ou Place cujo conteúdo é declarado lido. Os limites adicionais são superiores,
não escritas obrigatórias. Envelopes de extensão não contam nova execução.

### 10.3 Origem, cobertura e precisão — AIR 06

| Estrutura | Campos ou variantes |
| --- | --- |
| Origin | `written(id:OriginId,artifact:ArtifactId,location:Location?,includes:IncludeFrame[],exact:Bool)`; `derived(id:OriginId,inputs:OriginId[],rule:Text)`; `contractual(id:OriginId,authority:Text,version:Text)`; `unavailable(id:OriginId,reason:Text)` |
| Location | `line_columns(span:Span)`; `offsets(start:Natural,end:Natural,unit:Text,endExclusive:Bool)` |
| Position | `{line:Natural,column:Natural}` |
| Span | `{start:Position,end:Position,lineBase:Natural,columnBase:Natural,columnUnit:ColumnUnit,endExclusive:Bool}` |
| IncludeFrame | `{including:ArtifactId,included:ArtifactId,requestedName:Text,site:Location?}` |
| FactScope | `publication(publication:PublicationId)`; `unit(unit:UnitId)`; `entities(entities:Id[])` |
| Uncertainty | `{id:UncertaintyId,code:Text,dimensions:Dimension[],scope:FactScope,reason:Text,origin:OriginId}` |
| Coverage | `{inventory:InventoryStatus,scope:FactScope,items:CoverageItem[],uncertainties:UncertaintyId[]}` |
| CoverageItem | `{sourceKey:Text,origin:OriginId,status:CoverageStatus,outputs:Id[],uncertainties:UncertaintyId[],elimination:Elimination?}` |
| Elimination | `{rule:Text,origin:OriginId}` |
| Claim | `{scope:FactScope,status:PrecisionStatus,reasons:UncertaintyId[]}` |
| Precision | `{control:Claim,storage:Claim,effects:Claim,values:Claim,dependencies:Claim}` |

`Id` é qualquer domínio admitido na seção 4. Coordenadas preservam a base/unidade
publicada. `offsets` usa base zero e declara a unidade (por exemplo `octet`,
`unicode_scalar` ou `utf16_code_unit`; outra unidade exige interpretação explícita
negociada). Offsets conhecidos sem linha/coluna não exigem coordenadas fabricadas;
localização de origem não é intervalo de storage. `derived.inputs` é não vazio;
`unavailable` não é origem ausente. `Elimination` transporta regra e evidência da
eliminação já exigida por PROD-02/I-29; texto da regra não define operação nova.
`sourceKey`, mensagens e nomes de exibição são opacos ao consumidor semântico.

### 10.4 Tokens

Os tokens abaixo são escolhas lexicais para os conceitos AIR; não são nomes de enums
Java. Códigos de incerteza são os de AIR 06, §4, mais códigos qualificados permitidos.
Ações padrão são as de AIR 04, §7; categorias e relações, as de AIR 01, §§7–8.

| Tipo | Tokens |
| --- | --- |
| Dimension | `CONTROL, STORAGE, EFFECTS, VALUES, DEPENDENCIES` |
| PrecisionStatus | `EXACT, CONSERVATIVE, OPEN, UNAVAILABLE, NOT_APPLICABLE` |
| CoverageStatus | `MODELED, ABSTRACTED, UNSUPPORTED, INPUT_MISSING` |
| InventoryStatus | `COMPLETE, PARTIAL, UNAVAILABLE` |
| UnaryOperator | `NOT, NEG, TO_DECIMAL, LENGTH` |
| BinaryOperator | `EQ, NE, LT, LE, GT, GE, AND, OR, ADD, SUB, MUL, CONCAT` |
| Rounding | `TOWARD_ZERO, HALF_EVEN` |
| PassingMode | `VALUE, REFERENCE, COPY` |
| Lifetime | `ACTIVATION, PERSISTENT, EXTERNAL` |
| Visibility | `PRIVATE, SHARED, UNKNOWN` |
| ByteOrder | `LITTLE, BIG` |
| HaltKind | `NORMAL, ABNORMAL` |
| ColumnUnit | `UNICODE_SCALAR, UTF16_CODE_UNIT, OCTET` |
| OperandRole | `VALUE_READ, VALUE_WRITE, ADDRESS_READ, PREDICATE, CALL_TARGET, ARGUMENT_VALUE, ARGUMENT_REFERENCE, RESULT_TARGET, RESOURCE_TARGET, CONTROL_TARGET` |

## 11. Validação e campos desconhecidos

1. Verificar bytes, propriedades únicas, campos obrigatórios, formas e versões.
   Erro léxico/estrutural de JSON é `INPUT_ERROR`.
2. Materializar exatamente os fatos, sem inferir fatos faltantes. Referência
   quebrada, tipo contraditório ou outra violação AIR é `INVALID_IR`, com regra/site.
3. Validar as regras AIR implementadas e registrar obrigações/limites. Um schema
   de JSON não prova `sameDomain`, pureza, separação física ou verdade de lowering.
   `INCOMPLETE_VALIDATION` não é aprovação integral nem permissão para reparar fatos.
4. Negociar capacidades. Sem suporte preciso, usar fallback já presente e permitido
   ou emitir `UNSUPPORTED_CAPABILITY`; nunca ignorar operação/campo semântico.

Campos, discriminadores e tokens fechados não catalogados são rejeitados, sem
selecionar defaults. Versões não suportadas produzem incompatibilidade explícita.
Códigos/categorias qualificados só são aceitos onde a AIR autoriza extensão, com
negociação pertinente. O decoder não habilita desserialização arbitrária de runtime.

## 12. I/O e round-trip

O envelope contém uma Publication indivisível, com uma ou várias unidades. Publicações
diferentes não se juntam por coincidência de IDs locais. Streaming e índices privados
são permitidos; fechamento deve ser verificado antes de disponibilizar os fatos.
Limite de tempo/memória/cardinalidade é falha explícita, não prefixo válido.

`decode(encode(P))` preserva todos os fatos AIR, identidades, evidências, restantes e
ordens semânticas. Para documento aceito `J`, `encode(decode(J))` é a forma canônica
com os mesmos dados e ordem física de arrays; whitespace/ordem de propriedades não
são dados AIR. Igualdade semântica de dois decimais não apaga a representação
coeficiente/escala transportada. Nenhum dos percursos remove unknown em favor de null.

Um writer de arquivo pode escrever temporário e substituir o destino após escrita
completa, conforme garantias do filesystem. Essa escolha de I/O não muda o contrato
semântico e não autoriza entregar conteúdo parcialmente publicado.

## 13. Oráculos para promoção e codecs

- Escritas repetidas de publicação com mesma ordem física geram bytes idênticos;
  permutar inventários não cria nova ordem de execução.
- Round-trip conserva IDs completos, TypeRef, valores, origens, premissas, assinatura
  externa por site, efeitos, outcomes e todos os restantes.
- Dois sites com a mesma autoridade contratual mantêm posições/sujeitos separados;
  remover acesso à autoridade não altera a decodificação ou as observações.
- Inteiros e escalas acima de 2^53, bytes 00/FF, texto vazio, padding, Unicode
  suplementar, offsets de origem sem linha/coluna e ausência explícita sobrevivem
  sem limites de tipos de runtime.
- Propriedades duplicadas, `null` no lugar de unknown, campos omitidos, segundo
  documento, tokens privados de Validator e versões desconhecidas falham explicitamente.
- Contracasos de AIR O-82 a O-91 conservam diagnóstico/regra, inclusive escolha aberta,
  escopo errado, `ResourceId` como target, separação sem evidência e outcome duplicado.
- `copy_bytes` comum conserva `continue` em fallback sem terminador fictício;
  `opaque` não aceita esse fallthrough e conserva saltos/saídas/restantes.
- Consumo da mesma publicação construída em memória e decodificada produz observações
  equivalentes sob perfis/premissas idênticos; esta obrigação não implementa CFG/RD/values.
- Pelo menos duas implementações independentes verificam os casos de transporte antes
  de alegar interoperabilidade; conhecer Java não pode ser requisito de nenhuma delas.

Esses são requisitos a demonstrar, não resultados já obtidos. O handoff registra
as incompatibilidades da candidata anterior e o trabalho separado em `air-java`.
