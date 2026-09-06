# Analysis IR JSON Binding — proposta v1

**Status:** proposta para revisão e promoção em `analysis-ir`; não é contrato já
aprovado pelo simples fato de acompanhar `air-java`.

**Versão candidata do binding:** 1.0.0. **Semântica de referência:** AIR 2.0.0,
commit `0b2fbce7046010b22b32efa8cbc3e75ccba09442`.
**Destino sugerido:** `analysis-ir/bindings/json-v1.md`.

O documento especifica uma codificação de transporte, não uma nova IR. O significado
vem de [01](../especificacao/01-modelo-e-identidades.md),
[02](../especificacao/02-tipos-valores-e-operandos.md),
[04](../especificacao/04-operacoes.md) e
[06](../especificacao/06-incompletude-e-proveniencia.md).
Em conflito, a especificação semântica prevalece e este binding deve ser corrigido.

## 1. Objetivo e independência

O produtor e o consumidor podem trocar um arquivo UTF-8 e materializar a mesma
publicação imutável em memória. `cobol-lowering` e `analysis-cfg` usam adapters
externos ao core. O modelo `air-java` não importa JSON nem carrega anotações de
Jackson/Gson. Um adapter pode usar DTOs/mix-ins/factories próprios.

Não se serializam nomes de classes Java, `toString()`, identidade de objetos JVM,
callbacks, ponteiros, estados calculados de CFG, GEN/KILL, RD ou possible values.
Trocar arquivo por passagem direta de `Publication` não altera a porta do consumidor.

Esta proposta cobre o vocabulário listado abaixo, inclusive as três extensões
padronizadas. Payloads de novas extensões precisas precisam de revisão versionada
ou binding próprio negociado. Uma construção não interpretada pode usar `opaque`
com seus envelopes; não vira uma variante JSON arbitrária sem contrato.

## 2. Envelope

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
    "premises": [],
    "contracts": []
  }
}
```

O exemplo representa **zero conhecido no escopo declarado**, não uma análise
indisponível. Uma produção real não pode usar esse envelope para ocultar input.

`airVersion` é a única codificação de `Publication.airVersion`, localizada no
envelope. `bindingVersion` não é a versão da biblioteca Maven. `contracts` materializa
conteúdo explicitamente referenciado; não representa um serviço consultado depois.
Todos os campos estruturais do envelope são obrigatórios, mesmo com arrays vazios.

Uma versão major diferente não recebe interpretação automática. Esta candidata
aceita exatamente AIR 2.0.0; ampliar a matriz de compatibilidade exige evidência e
revisão do contrato. Não deduzir compatibilidade só porque os campos parecem iguais.

## 3. Regras léxicas e valores

O arquivo contém exatamente um objeto JSON, em UTF-8, sem BOM e sem conteúdo após
esse objeto além de whitespace permitido. Nomes de propriedades repetidos são
rejeitados; “último vence” não é recuperação válida. Strings representam escalares
Unicode; surrogates isolados são inválidos. Não normalizar Unicode, caixa ou espaços.

Inteiros matemáticos da AIR, coeficientes decimais, tamanhos físicos arbitrários e
literais inteiros são **strings decimais canônicas**, evitando perda em ferramentas
que usam IEEE-754: `"0"`, `"-17"`, `"9007199254740993"`. Não usar `-0`, `+1`, zeros
à esquerda ou notação exponencial. Escalas, posições e contagens Java limitadas são
números JSON inteiros exatos, com os limites documentados do campo, nunca doubles.

| Domínio literal | Forma |
| --- | --- |
| bool | `{"kind":"bool","value":true}` |
| int | `{"kind":"int","value":"123"}` |
| decimal | `{"kind":"decimal","coefficient":"125","scale":2}` |
| text | `{"kind":"text","value":"A "}`; vazio e padding são valores |
| bytes | `{"kind":"bytes","base64":"AP8="}`; RFC 4648 padrão, padding, sem whitespace |
| label | `{"kind":"label","label":<LabelId>,"domain":<LabelType>}` |

`decimal` conserva coeficiente e escala publicados; igualdade de valores em
dispatch é semântica, não comparação da codificação. `bytes` não é texto sem codec.
Não existe literal de `unknown_type`. Extensão de literal precisa de domínio,
manifesto e codificação próprios; não usar uma string genérica como valor arbitrário.

## 4. Identidades

Toda ocorrência de ID, inclusive referências, carrega domínio e namespace completo.
`localId` é string opaca, não nome COBOL nem expressão. As formas são:

```text
PublicationId = {domain:"publication", localId}
GlobalId      = {domain, publication:<PublicationId.localId>, localId}
UnitOwnedId   = {domain, publication:<PublicationId.localId>,
                 unit:<UnitId.localId>, localId}
OperandId     = {domain:"operand", publication, unit,
                 owner:{kind:"operation"|"entry", localId}, localId}
```

Domínios globais: `unit`, `storage`, `resource`, `artifact`, `origin`, `uncertainty`,
`premise`, `contract`, `relation`. Domínios pertencentes a unidade: `entry`, `label`,
`operation`, `object`, `completion_port`. Domínios não são intercambiáveis.

```json
{
  "domain": "operand",
  "publication": "p",
  "unit": "u",
  "owner": {"kind": "operation", "localId": "invoke-7"},
  "localId": "target"
}
```

É permitido repetir a codificação do mesmo ID como referência. Não é permitido
publicar duas entidades com esse mesmo ID no mesmo domínio. IDs são próprios da AIR;
correlação com Semantic Product pertence à proveniência, não à chave global.

## 5. União tipada, ausência e desconhecimento

Uma soma de variantes usa **`kind` explícito e obrigatório**. Não escolher variante
pela presença de campos, por nome da classe ou pelo texto de `observedKind`.

```json
{"kind":"known","type":{"kind":"text"}}
```

```json
{
  "kind":"unknown_type",
  "uncertainty":{"domain":"uncertainty","publication":"p","localId":"type-1"}
}
```

A referência acima deve apontar para `TYPE_UNKNOWN`. Domínio de extensão é
`{"kind":"opaque_type","name":"vendor.domain","version":"1.0.0"}`, dentro de
`known`; não é substituto para tipo desconhecido. `label` carrega unidade e universo.

Uma expressão desconhecida carrega seu próprio `typeRef`, `dependencies`,
`remainingReads` e razão **de valor**. A razão do valor não substitui `TYPE_UNKNOWN`.
Um predicado de valor desconhecido precisa continuar `known(bool)` para `branch`.

Campos opcionais são explicitamente `null`; arrays obrigatórios vazios são `[]`.
Campo ausente não vira `null`, zero, `false`, array vazio ou semântica `none`.
Limites usam variantes `none`/`within` explícitas; desconhecimento não é `none`.
`ContractKnowledge` contém exatamente um entre `contract` e `unknown`, o outro `null`.
Um `Entry` com corpo disponível não pode ter label ausente.

## 6. Ordenação e determinismo

A escrita canônica usa campos de cada objeto em ordem lexicográfica ordinal dos
nomes de propriedades. Arrays conservam a ordem determinística da publicação;
parâmetros, resultados, operandos, operações, condições de entrada e include chains
não são reordenados por nome. Nenhum consumidor extrai fluxo da posição das sequences.

Sem indentação e sem newline final na forma canônica. Escapar apenas aspas,
backslash e caracteres de controle; usar escapes curtos `\b`, `\f`, `\n`, `\r`,
`\t` quando aplicáveis e `\u00xx` minúsculo nos demais controles. Não escapar `/`
ou escalares Unicode imprimíveis. A representação é UTF-8, sem conversão ambiental.

Para a mesma publicação ordenada, versões e configuração, as codificações repetidas
são byte a byte idênticas. Isso não exige bytes iguais de duas IRs semanticamente
equivalentes com diferentes decomposições ou ordens físicas de sequences. Os
produtores continuam responsáveis por publicação deterministicamente ordenada.

Não incluir timestamp, duração, UUID aleatório, hash de objeto, thread ou informação
de execução no payload. `publicationId` não precisa ser um hash do próprio JSON e
não é chave persistente de programa. Não criar dependência circular ID↔digest.

## 7. Sequências, operações e pontos

```text
Sequence = {label, instructions:[Instruction...], terminator:Terminator, origin}
```

`instructions` nunca contém terminador e `terminator` contém exatamente um.
`invoke`, `opaque` e transferências locais/indiretas são terminadores. Não inserir
`jump` por proximidade física das sequences. `Operation.header` contém ID, origem,
coverage, precision e uncertainties; a unidade proprietária vem do ID completo e
a sequência proprietária vem da posição declarativa, validada por unicidade.

| kind | Campos específicos além de header |
| --- | --- |
| assign | destination, value |
| havoc.must | destination, reason |
| havoc.may | scope, reason |
| nop | nenhum |
| copy_bytes | destination, source, length, fallback |
| jump | destination |
| branch | predicate, trueDestination, falseDestination |
| dispatch | selector, cases, defaultDestination |
| invoke | action, target, arguments, results, effectBound, outcomes, contract, signatureGaps |
| return | values, entryScope |
| raise | tag, values |
| halt | haltKind (`NORMAL`/`ABNORMAL`) |
| opaque | observedKind, knownOperands, valueResults, envelope |
| local.invoke | entry, completionPorts, resume, fallback |
| local.boundary | port, defaultDestination, fallback |
| local.resume | fallback |
| local.unwind | count, destination, fallback |
| indirect.jump | target, within, fallback |

Os fallbacks pertencem à mesma operação; não se contam nem executam original e
fallback como eventos distintos. Capacidade necessária entra no manifesto.

Pontos usam `before`, `after`, `entry`, `exit`; identificam operação/entrada/unidade
e outcome conforme o caso. Nunca são inteiros de traversal ou números de linha.
Outcome usa `normal`, `exception` com tag, `other_exception`, `halt`, `diverge`.

## 8. Expressões, locais, interações e envelopes

| Soma | Discriminadores e campos |
| --- | --- |
| Expression | `literal(header,value)`; `read(header,place)`; `unknown(header,typeRef,dependencies,remainingReads,reason)` |
| Expression composta | `unary(header,operator,argument)`; `binary(header,operator,left,right)`; `quantize(header,value,scale,rounding)`; `fit_text(header,value,length,pad)`; `slice_text(header,value,start,count,boundsProof)`; `trim_right(header,value,characters)` |
| Place | `object(header,object)`; `choice(header,candidates,remainder,typeRef,knownRemainderDomainProof)`; `region_slice(header,region,offset,length,codec,typeRef,accessProof)` |
| Target | `internal(entry)`; `literal(category,namespace,name,namePolicy,origin)`; `computed(category,namespace,name,namePolicy,origin)`; `resource(resource)` |
| Argument | `value(value)`; `copy(value)`; `reference(place)` |
| NamePolicy | `exact`; `contract(contract)`; `unknown(uncertainty)` |
| Codec | `bytes.identity`; `text.ascii`; `binary(signed,width,order)`; `extension(name,version,logicalType,contract)`; `unknown(logicalType,reason)` |
| StorageBinding | `cell(storage)`; `view(region,offset,extent,codec)`; `alias(object)`; `alternatives(alternatives,remainder)`; `unknown(scope,reason)` |
| Storage | `cell(header,typeRef)`; `region(header,extent,extentUnknown)` |

A notação da tabela descreve nomes de campos, não uma sintaxe adicional a implementar.
Cada objeto de variante tem `kind`. Os outros nomes são suas propriedades JSON.
Operadores, roles, status e demais enumerações fechadas usam os nomes explícitos do
catálogo abaixo; valores não reconhecidos são erro de compatibilidade, não default.

Um `Operand.header` contém `id`, `role` e `origin`. O tipo pode ser derivado por regra
normativa, por exemplo do objeto em `object` ou do valor em `literal`, sem anotar um
segundo tipo que contradiga a fonte canônica.

`Envelope = {memory, control, dependencies}`. O envelope de memória contém
`knownReads`, `otherReads`, `knownWrites`, `otherWrites`, `mustOverwrite`. As listas
conhecidas referem **OperandId** presentes naquela operação, não nomes de variáveis.
Escrita conhecida/must identifica Place. Isso evita duplicar a ocorrência só para
referi-la na anotação do envelope.

`ControlEnvelope = {known, remainder}`. Alternativas: `normal(label)`,
`exception(tag,destination)`, `any_exception(destination)`, `halt`, `diverge`.
Destination excepcional é `handler(label)` ou `propagate`. Um `invoke` admite no
máximo um normal e tags únicas; um `opaque` pode admitir vários destinos conhecidos.

`DependencyEnvelope = {known:[{action,target,point,origin}], remainder}`.
`ForeignEffects = {reads,writes,mustOverwrite}`;
`EffectBound = {otherwise,perOutcome:[{outcome,effects}]}`. Ausência de contrato não
cria efeitos vazios; argumentos/resultados/outcomes incompletos mantêm suas lacunas.

Bounds: memória e controle usam `none` ou `within(scope)`. Scope de memória usa
`objects(objects)`, `storage(storage)`, `visible(unit,includingExternal)`,
`all(publication,includingEnvironment)`, `union(members)`. Scope de controle usa
`labels(labels)`, `unit(unit,labels,normalExit,exceptionalExit,halt,diverge,externalControl)`,
`all(publication)`, `union(members)`. Recursos usam `none`, `any_resource` ou
`categories(categories)`. Não substituir um scope por string de mensagem.

## 9. Premissas e provas de domínio

`Premise = {id,authority,justification,origin,assertion}`.
`SameDomain` é `{"kind":"same_domain","left":<Subject>,"right":<Subject>,"scope":<DomainProofScope>}`.
Não é uma propriedade inferida de IDs iguais de incerteza.

Sujeitos: `object(object)`, `cell(cell)`, `operand(operand)`,
`parameter(entry,position)`, `result(entry,position)`,
`call_parameter(invocation,signature,position)`, `call_result(invocation,signature,position)`.
`signature` é `entry(entry)` ou `external(contract)`.

Escopos fechados: `publication`, `unit(unit)`, `entry(entry)`,
`operation(operation)`, `invocation(invocation)`, `intersection(left,right)`.
Interseção vazia é válida, mas não prova nada em site algum. Não há `activation`.
Vínculos de assinatura são específicos de cada site, sem misturar chamador/chamado.

Uma premissa sobre a ocorrência inteira de uma escolha aberta cobre candidatos e
todo o restante. Não remover esse restante, promover o tipo ou escolher candidato.

Esta representação também admite `disjoint_storage(storage,scope)` e
`safety(property,subjects,scope)` como formas tipadas de premissas. São alegações
com autoridade e origem, não semântica derivada pelo decoder. `safety` apenas
identifica a evidência de precondições; sua verdade precisa ser estabelecida fora
do parser/validator estrutural. A interpretação dessas formas deve ser revisada
junto deste binding antes de sua promoção; não são licença para inventar premissas.

### 9.1 Estados de entrada e relações de artefato

`InitialValue` usa `literal(value)`, `parameter(position)`, `preserve`,
`external_unknown(reason)` ou `uninitialized(reason)`. Na primeira forma, `value`
é uma ocorrência `Expression.literal` pertencente à entrada, com seu próprio
OperandId e origem. A associação Place de cada condição também pertence à entrada.
`preserve` não equivale a reaplicar inicialização numa back-edge.

`RelationTarget` usa `internal(artifact)` ou `external(resource)`. Uma relação de
artefato não possui ponto de execução, e não é serializada como `invoke`.

## 10. Origem, cobertura e enumerações

Origem: `written(id,artifact,span,includes,exact)`,
`derived(id,inputs,rule)`, `contractual(id,authority,version)`,
`unavailable(id,reason)`. `Span` contém início/fim, base de linha/coluna,
unidade de coluna e se o fim é exclusivo. IncludeFrame contém artefatos e site,
sem exigir que o consumidor reabra arquivos. Origem derivada nunca recebe span inventado.

Coverage: `{inventory,scope,items,uncertainties}`. Item:
`{sourceKey,origin,status,outputs,uncertainties,elimination}`.
Eliminação é `null` ou `{justification:<PremiseId>,rule}`. `sourceKey` é opaco ao
consumidor; não é chave para reconstruir semântica de origem.

Precision contém `control`, `storage`, `effects`, `values`, `dependencies`.
Cada claim contém `{scope,status,reasons}`. Scope de fatos:
`publication(publication)`, `unit(unit)`, `entities(entities)`.

Uncertainty contém `{id,code,dimensions,scope,reason,origin}`; mensagem humana
não governa comportamento. `INPUT_MISSING` não substitui valor externo desconhecido.

Os campos estruturados restantes seguem o catálogo de campos abaixo, com ID,
Optional, lista e valor codificados pelas regras anteriores. O catálogo descreve
a forma abstrata do binding, não exige classes ou frameworks Java.

## 11. Validação e erros nos adapters

1. Ler bytes e verificar UTF-8, um único documento, propriedades únicas, nomes,
   campos, variantes e versões. Erro físico/léxico é `INPUT_ERROR`.
2. Decodificar DTOs de transporte sem inventar informação e materializar o modelo.
   Forma semântica impossível e referência inválida são `INVALID_IR`, com regra e site.
3. Validar fechamento, tipos/provas, envelopes e demais regras implementadas da AIR.
   `INCOMPLETE_VALIDATION` não deve virar aprovação silenciosa.
4. Negociar as capacidades que o consumidor realmente interpreta. Ausência de
   suporte é incompatibilidade/consumo conservador explícito, nunca `nop`.

Versão/variante não suportada não deve ser reparada ou descartada. Campos extras
não previstos nesta candidata são rejeitados; adições compatíveis precisam de regra
explícita em revisão do binding. Um future extension adapter deve ser negociado,
não ativado por nomes de propriedades. Não habilitar desserialização de classes
arbitrárias nem default typing baseado em nomes JVM.

O schema JSON futuro poderá verificar forma. Ele não substituirá `sameDomain`,
fechamento, relações de namespace ou a verdade semântica das premissas.

## 12. I/O, escala e publicação

O arquivo pode conter uma ou mais unidades da mesma Publication; não concatenar
publicações com IDs locais. Para corpus grande, usar múltiplos arquivos/publicações
com escopos explícitos, não um documento global obrigatório. Um adapter pode
streamar o JSON e materializar índices, respeitando fechamento antes de entregar
a publicação ao core. Limites atingidos são falha/limitação explícita, não prefixo válido.

Writer de arquivo deve produzir temporário e substituir o destino somente após
escrita completa, quando essa garantia de filesystem for disponível. Um consumidor
não deve abrir um payload parcialmente escrito como se fosse publicação fechada.

## 13. Oráculos antes de aprovar os adapters

- Duas escritas da mesma publicação e duas construções independentes equivalentes
  no contrato ordenado produzem os mesmos bytes.
- Decodificação preserva todos os tipos/valores/IDs/outcomes/uncertainties/premises.
- Origem exata, tipo conhecido e domínio comum não promovem outras dimensões.
- Operações omitidas, referências trocadas, reason perdido ou empty substituindo
  unknown são detectados.
- Place/choice/alias e `sameDomain` têm contracasos com restante aberto e escopo errado.
- Inteiros além de 2^53, bytes 00/FF, texto vazio, padding, escala decimal, Unicode
  suplementar e estados opcionais sobrevivem ao round trip.
- Propriedades duplicadas, trailing document, major desconhecida, variantes extras
  e enums não reconhecidos falham explicitamente.
- O mesmo BuildCfg sobre publicação construída em memória e decodificada do arquivo
  produz resultado semanticamente equivalente, sob as mesmas opções/publicação.
- Alterar adapter não cria dependência JSON em `air-java`, lowerer core ou CFG core.

**Nenhum desses oráculos de codec é anunciado como executado por `air-java`: a
biblioteca entregue deliberadamente não contém codec.** Esta é a obrigação dos
adapters a implementar após aprovação do binding.

## Apêndice A — Campos estruturados

As tabelas usam nomes curtos de estruturas; a coluna de referência Java é somente
um mapa de implementação. Os discriminadores acima prevalecem. IDs usam §4; tipos,
valores e variantes usam §§3–10. `Optional<X>` significa campo explícito `null` ou X.
A versão da publicação sai no envelope (`airVersion`), não duplicada no objeto interno.

| Estrutura / referência Java informativa | Campos |
| --- | --- |
| `Artifacts.InternalArtifact` | `ArtifactId artifact` |
| `Artifacts.ExternalArtifact` | `Interactions.LiteralTarget resource` |
| `Artifacts.Relation` | `RelationId id, ArtifactId source, RelationTarget destination, String kind, OriginId origin, Evidence.CoverageStatus coverage` |
| `Capabilities.Capability` | `String name, int major` |
| `Capabilities.Manifest` | `List<Capability> required, List<Capability> provided` |
| `Control.Handler` | `LabelId label` |
| `Control.Normal` | `LabelId label` |
| `Control.Exceptional` | `String tag, ExceptionDestination destination` |
| `Control.AnyException` | `ExceptionDestination destination` |
| `Control.Envelope` | `List<Alternative> known, Scopes.ControlBound remainder` |
| `Control.ExceptionOutcome` | `String tag` |
| `Control.Before` | `OperationId operation` |
| `Control.After` | `OperationId operation, OutcomeKey outcome` |
| `Control.EntryPoint` | `EntryId entry` |
| `Control.ExitPoint` | `UnitId unit, OutcomeKey outcome` |
| `Entries.CompletionPort` | `CompletionPortId id, OriginId origin` |
| `Entries.LiteralInitial` | `Expressions.Literal value` |
| `Entries.ParameterInitial` | `int position` |
| `Entries.ExternalUnknown` | `UncertaintyId reason` |
| `Entries.Uninitialized` | `UncertaintyId reason` |
| `Entries.InitialCondition` | `Place place, InitialValue value, OriginId origin, List<PremiseId> premises` |
| `Entries.EntryState` | `List<InitialCondition> conditions, List<UncertaintyId> uncertainties` |
| `Entries.Entry` | `EntryId id, Optional<LabelId> initialLabel, Interactions.Signature signature, EntryState state, OriginId origin` |
| `Envelopes.MemoryEnvelope` | `List<OperandId> knownReads, Scopes.MemoryBound otherReads, List<OperandId> knownWrites, Scopes.MemoryBound otherWrites, List<OperandId> mustOverwrite` |
| `Envelopes.ResourceUse` | `String action, Interactions.Target target, Control.ProgramPoint point, OriginId origin` |
| `Envelopes.DependencyEnvelope` | `List<ResourceUse> known, Scopes.DependencyBound remainder` |
| `Envelopes.Envelope` | `MemoryEnvelope memory, Control.Envelope control, DependencyEnvelope dependencies` |
| `Evidence.Claim` | `Scopes.FactScope scope, PrecisionStatus status, List<UncertaintyId> reasons` |
| `Evidence.Precision` | `Claim control, Claim storage, Claim effects, Claim values, Claim dependencies` |
| `Evidence.Uncertainty` | `UncertaintyId id, String code, List<Dimension> dimensions, Scopes.FactScope scope, String reason, OriginId origin` |
| `Evidence.Elimination` | `PremiseId justification, String rule` |
| `Evidence.CoverageItem` | `String sourceKey, OriginId origin, CoverageStatus status, List<Id> outputs, List<UncertaintyId> uncertainties, Optional<Elimination> elimination` |
| `Evidence.Coverage` | `InventoryStatus inventory, Scopes.FactScope scope, List<CoverageItem> items, List<UncertaintyId> uncertainties` |
| `Expressions.Literal` | `Operand.Header header, Values.LiteralValue value` |
| `Expressions.Read` | `Operand.Header header, Place place` |
| `Expressions.Unknown` | `Operand.Header header, Types.TypeRef typeRef, List<Expression> dependencies, Scopes.MemoryBound remainingReads, UncertaintyId reason` |
| `Expressions.Unary` | `Operand.Header header, UnaryOperator operator, Expression argument` |
| `Expressions.Binary` | `Operand.Header header, BinaryOperator operator, Expression left, Expression right` |
| `Expressions.Quantize` | `Operand.Header header, Expression value, int scale, Rounding rounding` |
| `Expressions.FitText` | `Operand.Header header, Expression value, BigInteger length, String pad` |
| `Expressions.SliceText` | `Operand.Header header, Expression value, Expression start, Expression count, Optional<PremiseId> boundsProof` |
| `Expressions.TrimRight` | `Operand.Header header, Expression value, String characters` |
| `Interactions.ContractName` | `ContractId contract` |
| `Interactions.UnknownName` | `UncertaintyId uncertainty` |
| `Interactions.InternalTarget` | `EntryId entry` |
| `Interactions.LiteralTarget` | `String category, String namespace, String name, NamePolicy namePolicy, OriginId origin` |
| `Interactions.ComputedTarget` | `String category, String namespace, Expression name, NamePolicy namePolicy, OriginId origin` |
| `Interactions.ResourceTarget` | `ResourceId resource` |
| `Interactions.Resource` | `ResourceId id, LiteralTarget description` |
| `Interactions.ValueArgument` | `Expression value` |
| `Interactions.CopyArgument` | `Expression value` |
| `Interactions.ReferenceArgument` | `Place place` |
| `Interactions.Parameter` | `int position, PassingMode mode, Types.TypeRef typeRef, Optional<ObjectId> object, OriginId origin` |
| `Interactions.ResultSlot` | `int position, Types.TypeRef typeRef, OriginId origin` |
| `Interactions.Signature` | `List<Parameter> parameters, List<ResultSlot> results, Optional<UncertaintyId> incomplete` |
| `Interactions.ForeignEffects` | `Scopes.MemoryBound reads, Scopes.MemoryBound writes, List<OperandId> mustOverwrite` |
| `Interactions.OutcomeEffects` | `Control.OutcomeKey outcome, ForeignEffects effects` |
| `Interactions.EffectBound` | `ForeignEffects otherwise, List<OutcomeEffects> perOutcome` |
| `Interactions.Contract` | `ContractId id, String authority, SemanticVersion version, Signature signature, Optional<EffectBound> effects, List<PremiseId> premises, List<UncertaintyId> uncertainties, OriginId origin` |
| `Interactions.ContractKnowledge` | `Optional<ContractId> contract, Optional<UncertaintyId> unknown` |
| `Interactions.EntrySignature` | `EntryId entry` |
| `Interactions.ExternalSignature` | `ContractId contract` |
| `Memory.BinaryCodec` | `boolean signed, int width, ByteOrder order` |
| `Memory.ExtensionCodec` | `String name, SemanticVersion version, Types.TypeRef logicalType, ContractId contract` |
| `Memory.UnknownCodec` | `Types.TypeRef logicalType, UncertaintyId reason` |
| `Memory.CellBinding` | `StorageId storage` |
| `Memory.ViewBinding` | `StorageId region, BigInteger offset, BigInteger extent, Codec codec` |
| `Memory.AliasBinding` | `ObjectId object` |
| `Memory.AlternativesBinding` | `List<Binding> alternatives, Scopes.MemoryBound remainder` |
| `Memory.UnknownBinding` | `Scopes.MemoryScope scope, UncertaintyId reason` |
| `Memory.StorageHeader` | `StorageId id, Optional<UnitId> owner, Lifetime lifetime, Visibility visibility, OriginId origin` |
| `Memory.Cell` | `StorageHeader header, Types.TypeRef typeRef` |
| `Memory.Region` | `StorageHeader header, Optional<BigInteger> extent, Optional<UncertaintyId> extentUnknown` |
| `Memory.ObjectDeclaration` | `ObjectId id, Optional<String> displayName, Types.TypeRef typeRef, Binding storage, Visibility visibility, OriginId origin, Evidence.CoverageStatus coverage, Evidence.Precision precision` |
| `Memory.ByteRange` | `StorageId region, Expression offset, Expression extent, Optional<PremiseId> boundsProof` |
| `Operations.Header` | `OperationId id, OriginId origin, Evidence.CoverageStatus coverage, Evidence.Precision precision, List<UncertaintyId> uncertainties` |
| `Operations.Assign` | `Header header, Place destination, Expression value` |
| `Operations.HavocMust` | `Header header, Place destination, UncertaintyId reason` |
| `Operations.HavocMay` | `Header header, Scopes.MemoryScope scope, UncertaintyId reason` |
| `Operations.Nop` | `Header header` |
| `Operations.CopyBytes` | `Header header, Memory.ByteRange destination, Memory.ByteRange source, BigInteger length, Envelopes.Envelope fallback` |
| `Operations.Jump` | `Header header, LabelId destination` |
| `Operations.Branch` | `Header header, Expression predicate, LabelId trueDestination, LabelId falseDestination` |
| `Operations.Case` | `Values.LiteralValue value, LabelId destination` |
| `Operations.Dispatch` | `Header header, Expression selector, List<Case> cases, LabelId defaultDestination` |
| `Operations.Invoke` | `Header header, String action, Interactions.Target target, List<Interactions.Argument> arguments, List<Place> results, Interactions.EffectBound effectBound, Control.Envelope outcomes, Interactions.ContractKnowledge contract, List<UncertaintyId> signatureGaps` |
| `Operations.Return` | `Header header, List<Expression> values, List<EntryId> entryScope` |
| `Operations.Raise` | `Header header, String tag, List<Expression> values` |
| `Operations.Halt` | `Header header, HaltKind haltKind` |
| `Operations.Opaque` | `Header header, String observedKind, List<Operand> knownOperands, List<Place> valueResults, Envelopes.Envelope envelope` |
| `Operations.LocalInvoke` | `Header header, LabelId entry, List<CompletionPortId> completionPorts, LabelId resume, Envelopes.Envelope fallback` |
| `Operations.LocalBoundary` | `Header header, CompletionPortId port, LabelId defaultDestination, Envelopes.Envelope fallback` |
| `Operations.LocalResume` | `Header header, Envelopes.Envelope fallback` |
| `Operations.LocalUnwind` | `Header header, int count, LabelId destination, Envelopes.Envelope fallback` |
| `Operations.IndirectJump` | `Header header, Expression target, Types.LabelType within, Envelopes.Envelope fallback` |
| `Origins.Position` | `int line, int column` |
| `Origins.Span` | `Position start, Position end, int lineBase, int columnBase, ColumnUnit columnUnit, boolean endExclusive` |
| `Origins.IncludeFrame` | `ArtifactId including, ArtifactId included, String requestedName, Optional<Span> site` |
| `Origins.Written` | `OriginId id, ArtifactId artifact, Optional<Span> span, List<IncludeFrame> includes, boolean exact` |
| `Origins.Derived` | `OriginId id, List<OriginId> inputs, String rule` |
| `Origins.Contractual` | `OriginId id, String authority, String version` |
| `Origins.Unavailable` | `OriginId id, String reason` |
| `Origins.Artifact` | `ArtifactId id, String logicalName, Optional<String> contentDigest` |
| `Places.ObjectPlace` | `Operand.Header header, ObjectId object` |
| `Places.Choice` | `Operand.Header header, List<Place> candidates, Scopes.MemoryBound remainder, Types.TypeRef typeRef, Optional<PremiseId> knownRemainderDomainProof` |
| `Places.RegionSlice` | `Operand.Header header, StorageId region, Expression offset, Expression length, Memory.Codec codec, Types.TypeRef typeRef, Optional<PremiseId> accessProof` |
| `Proofs.ObjectDomain` | `ObjectId object` |
| `Proofs.CellDomain` | `StorageId cell` |
| `Proofs.OperandDomain` | `OperandId operand` |
| `Proofs.ParameterDomain` | `EntryId entry, int position` |
| `Proofs.ResultDomain` | `EntryId entry, int position` |
| `Proofs.CallParameterDomain` | `OperationId invocation, Interactions.SignatureTarget signature, int position` |
| `Proofs.CallResultDomain` | `OperationId invocation, Interactions.SignatureTarget signature, int position` |
| `Proofs.UnitDomain` | `UnitId unit` |
| `Proofs.EntryDomain` | `EntryId entry` |
| `Proofs.OperationDomain` | `OperationId operation` |
| `Proofs.InvocationDomain` | `OperationId invocation` |
| `Proofs.Intersection` | `DomainProofScope left, DomainProofScope right` |
| `Proofs.EntrySite` | `EntryId entry` |
| `Proofs.OperationSite` | `OperationId operation` |
| `Proofs.InvocationSite` | `OperationId invocation` |
| `Proofs.SameDomain` | `DomainSubject left, DomainSubject right, DomainProofScope scope` |
| `Proofs.DisjointStorage` | `List<StorageId> storage, Scopes.FactScope scope` |
| `Proofs.SafetyAssertion` | `SafetyProperty property, List<Id> subjects, DomainProofScope scope` |
| `Proofs.Premise` | `PremiseId id, String authority, String justification, OriginId origin, Assertion assertion` |
| `Publication` | `PublicationId id, SemanticVersion airVersion, Capabilities.Manifest capabilities, List<Origins.Artifact> artifacts, List<Unit> units, List<Memory.Storage> storage, List<Interactions.Resource> resources, List<Artifacts.Relation> artifactRelations, List<Origins.Origin> origins, Evidence.Coverage coverage, List<Evidence.Uncertainty> uncertainties, List<Proofs.Premise> premises, List<Interactions.Contract> contracts` |
| `Scopes.PublicationScope` | `PublicationId publication` |
| `Scopes.UnitScope` | `UnitId unit` |
| `Scopes.EntityScope` | `List<Id> entities` |
| `Scopes.ObjectsMemory` | `List<ObjectId> objects` |
| `Scopes.StorageMemory` | `List<StorageId> storage` |
| `Scopes.VisibleMemory` | `UnitId unit, boolean includingExternal` |
| `Scopes.AllMemory` | `PublicationId publication, boolean includingEnvironment` |
| `Scopes.MemoryUnion` | `List<MemoryScope> members` |
| `Scopes.WithinMemory` | `MemoryScope scope` |
| `Scopes.LabelsControl` | `List<LabelId> labels` |
| `Scopes.UnitControl` | `UnitId unit, boolean labels, boolean normalExit, boolean exceptionalExit, boolean halt, boolean diverge, boolean externalControl` |
| `Scopes.AllControl` | `PublicationId publication` |
| `Scopes.ControlUnion` | `List<ControlScope> members` |
| `Scopes.WithinControl` | `ControlScope scope` |
| `Scopes.ResourceCategories` | `List<String> categories` |
| `Sequence` | `LabelId label, List<Instruction> instructions, Terminator terminator, OriginId origin` |
| `Types.ExtensionType` | `String name, SemanticVersion version` |
| `Types.LabelType` | `UnitId unit, List<LabelId> labels` |
| `Types.Known` | `Type type` |
| `Types.UnknownType` | `UncertaintyId uncertainty` |
| `Unit` | `UnitId id, Optional<UnitId> containingUnit, List<Memory.ObjectDeclaration> objects, List<ObjectId> visibleObjects, List<Entries.Entry> entries, List<Sequence> sequences, List<Entries.CompletionPort> completionPorts, BodyAvailability body, Optional<UncertaintyId> bodyUnavailable, Evidence.Coverage coverage, OriginId origin` |
| `Values.BoolValue` | `boolean value` |
| `Values.IntValue` | `BigInteger value` |
| `Values.DecimalValue` | `BigInteger coefficient, int scale` |
| `Values.TextValue` | `String value` |
| `Values.BytesValue` | `List<Integer> octets` |
| `Values.LabelValue` | `LabelId label, Types.LabelType domain` |

## Apêndice B — Tokens de enumerações

Os enums unitários usados como variante (`NoMemory`, `NoControl`, `Propagate` etc.)
usam os discriminadores descritos acima, não a palavra `INSTANCE`.

| Referência informativa | Tokens |
| --- | --- |
| `Evidence.Dimension` | `CONTROL, STORAGE, EFFECTS, VALUES, DEPENDENCIES` |
| `Evidence.PrecisionStatus` | `EXACT, CONSERVATIVE, OPEN, UNAVAILABLE, NOT_APPLICABLE` |
| `Evidence.CoverageStatus` | `MODELED, ABSTRACTED, UNSUPPORTED, INPUT_MISSING` |
| `Evidence.InventoryStatus` | `COMPLETE, PARTIAL, UNAVAILABLE` |
| `Expressions.UnaryOperator` | `NOT, NEG, TO_DECIMAL, LENGTH` |
| `Expressions.BinaryOperator` | `EQ, NE, LT, LE, GT, GE, AND, OR, ADD, SUB, MUL, CONCAT` |
| `Expressions.Rounding` | `TOWARD_ZERO, HALF_EVEN` |
| `Interactions.PassingMode` | `VALUE, REFERENCE, COPY` |
| `Memory.Lifetime` | `ACTIVATION, PERSISTENT, EXTERNAL` |
| `Memory.Visibility` | `PRIVATE, SHARED, UNKNOWN` |
| `Memory.ByteOrder` | `LITTLE, BIG` |
| `Operations.HaltKind` | `NORMAL, ABNORMAL` |
| `Origins.ColumnUnit` | `UNICODE_SCALAR, UTF16_CODE_UNIT, OCTET` |
| `Proofs.SafetyProperty` | `VALID_PURE_ACCESS, VALID_TEXT_SLICE, VALID_CODEC_WRITE, CHOICE_REMAINDER_DOMAIN, EXTENSION_EQUALITY_DEFINED` |
| `Unit.BodyAvailability` | `AVAILABLE, UNAVAILABLE` |
