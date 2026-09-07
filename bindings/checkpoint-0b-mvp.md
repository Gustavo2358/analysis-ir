# Checkpoint 0B — Baseline AIR JSON e conformidade mínima do MVP

**Informativo — base de implementação e plano de evidência, não outro binding.**
O futuro módulo separado de codec JSON no repositório `air-java` implementará a
candidata pinada abaixo. `air-json` é seu nome conceitual/provisório; artifactId,
coordenadas e topologia Maven serão decididos no discovery 0C-D.
O primeiro E2E exercitará apenas as formas transitivamente necessárias à Publication
do slice GOBACK. Nenhum encode, decode, round-trip ou E2E é anunciado como executado
por este checkpoint documental.

## 1. Baseline imutável

| Registro | Valor |
| --- | --- |
| analysis-ir baseline SHA | `122ce54e1b9ef9b00646f93ece409ca8b63bc933` |
| Origem do pin | `main` limpa, checkout de `main` e `git pull --ff-only` no início de 0B; resultado: `Already up to date.` |
| Binding document | [`bindings/json-v1.md` no SHA pinado](https://github.com/Gustavo2358/analysis-ir/blob/122ce54e1b9ef9b00646f93ece409ca8b63bc933/bindings/json-v1.md) |
| `binding` | `analysis-ir-json` |
| `bindingVersion` | `1.0.0` |
| `airVersion` | `2.0.0` |
| Status normativo do binding | **DRAFT — não NORMATIVE / ACCEPTED** |
| Estado da AIR | Edição 2.0.0 em fechamento antes de publicação estabilizada; documentos de `especificacao/` e `conformidade/` normativos |
| Data do snapshot | `2026-09-07T12:24:30-03:00` (America/Sao_Paulo) |
| Branch de 0B | `docs/air-json-mvp-baseline`, criada a partir desse SHA |

O SHA coincide com o snapshot observado no roadmap. O arquivo solicitado como
`handoff-roadmap-integracao-cobol-air-cfg.md` foi localizado no workspace sob o nome
`roadmap.md`, título **Handoff / Roadmap de Integração — COBOL → Semantic Product →
AIR → CFG**, data de referência 7 de setembro de 2026. Sua seção 6, **Checkpoint 0B
— analysis-ir**, governa o escopo; §§1.2, 1.3 e 1.6 registram as decisões de pin,
codec compartilhado e hardening adiado. SHA-256 do arquivo consultado:
`9d92d3fd4e0c6ffce8c5d2d0adc1caae32c15c70cb46cb5b5a717eba657a9e32`.
Esse arquivo externo não foi alterado. A instrução desta sessão limita a entrega
a PR pronto para revisão humana, sem o merge previsto no acompanhamento geral.

**Implemente o SHA acima, não uma `main` flutuante.** Por exemplo, dentro de
`analysis-ir`, `git show 122ce54e1b9ef9b00646f93ece409ca8b63bc933:bindings/json-v1.md`
recupera a candidata.

Os links relativos servem apenas para navegação no checkout corrente. Para
implementação e decisões normativas, o binding e todo conteúdo de referência em
`bindings/`, `especificacao/` e `conformidade/` devem ser resolvidos no baseline
`122ce54e1b9ef9b00646f93ece409ca8b63bc933`, por exemplo com
`git show <SHA>:<path>`. Uma revisão posterior da `main` não altera este baseline
nem autoriza combinar o binding pinado com normativos de outra revisão.
O commit de documentação de 0B não substitui o pin nem cria uma versão. A adoção
de outro baseline exige novo registro e avaliação explícita de diferenças.

A hierarquia permanece AIR → binding → implementação. As autoridades de leitura
são [escopo AIR](../especificacao/00-escopo-e-convencoes.md),
[modelo/identidades](../especificacao/01-modelo-e-identidades.md),
[operações](../especificacao/04-operacoes.md),
[controle](../especificacao/05-controle-e-invocacoes.md),
[proveniência/incompletude](../especificacao/06-incompletude-e-proveniencia.md),
[invariantes](../conformidade/01-invariantes.md) e os demais normativos referidos
pelo [binding](json-v1.md). A [revisão da candidata](revisao-json-v1.md) explica a
reconciliação; a [política §5.2](../especificacao/09-extensibilidade-e-compatibilidade.md#52-fechamento-do-modelo-antes-da-estabilização-da-200)
explica o estado pré-estabilização e a ausência de compatibilidade prometida entre
candidatas anteriores. Merge, versão Maven e sucesso de um Validator não promovem
o binding.

## 2. Evidência do slice e limites da observação

Foram consultados somente em leitura:

| Fonte | Snapshot e evidência |
| --- | --- |
| `cobol-lower` | `e2488a362478057de7d59cdf9ae2b38b1f4040d3`, working tree limpa em `main`; [regra do slice](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/docs/domain/first-slice-entry-goback.md), [EntryGobackLowerer](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/core/src/main/java/io/github/gustavo2358/lower/application/EntryGobackLowerer.java) e [SourceOrigins](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/core/src/main/java/io/github/gustavo2358/lower/application/SourceOrigins.java) |
| Testemunho SP existente | No mesmo SHA do lower: [fixture `cobol-semantic-product.json`](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/adapters/src/test/resources/sp/cobol-semantic-product.json), SP `1.1.0`, `AIR-FIRST`; [oracle FIRST-LOWER](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/docs/evals/first-slice-oracle.md) e [LoweringSuite](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/core/src/test/java/io/github/gustavo2358/lower/testing/LoweringSuite.java) |
| `air-java` | `f1973c8f7bfa85ad3f433e172a1d3c7de2c79994`, working tree limpa em `chore/air-java-harness`; [modelo](https://github.com/Gustavo2358/air-java/tree/f1973c8f7bfa85ad3f433e172a1d3c7de2c79994/src/main/java/io/github/gustavo2358/air/model), [source lock](https://github.com/Gustavo2358/air-java/blob/f1973c8f7bfa85ad3f433e172a1d3c7de2c79994/docs/sources.lock.json) e [reconciliação](https://github.com/Gustavo2358/air-java/blob/f1973c8f7bfa85ad3f433e172a1d3c7de2c79994/docs/reconciliation-air-2.md) |

O [source lock do lower](https://github.com/Gustavo2358/cobol-lower/blob/e2488a362478057de7d59cdf9ae2b38b1f4040d3/docs/sources/sources.lock.json)
pina `air-java` em `6a4091e5394fc22b3d2ada9abbdb530eb3572a58`. O diff desse SHA
para o checkout Java consultado é vazio em `src/main/java/io/github/gustavo2358/air/model`.
Ambos os locks referem o baseline AIR desta seção 1. Esses dados confirmam o shape
da implementação; não fornecem autoridade para acrescentar campos ao binding.

O mapeamento abaixo resulta da leitura da fixture, do caminho de produção e das
assertivas existentes, **sem executar frontend, lower ou builds de outros repos**.
Não é um golden AIR JSON produzido por codec. O E2E oficial ainda deverá gerar e
consumir bytes reais pelo frontend/lower/codec/CFG nos checkpoints posteriores.

### 2.1 Publication exigida pelo GOBACK

A admissão atual exige uma entrada primária conhecida, start fechado no GOBACK
único, zero DATA/parâmetros e ausência de RETURNING conhecida. A parcialidade do
inventário de entradas alternativas é conservada. O nome `AIR-FIRST` não participa
da regra. O mapeamento é de `CURRENT_PROGRAM_INVOCATION`/`NONE` para `return`, com
saída da ativação corrente e sem sucessor local ([AIR 04 §8](../especificacao/04-operacoes.md#8-saídas)).

Esta tabela descreve valores exercitados, não restringe cardinalidades da AIR:

| Forma / caminho | Conteúdo do testemunho e fechamento transitivo | Binding |
| --- | --- | --- |
| Envelope + Publication | Um objeto com `binding`, `bindingVersion`, `airVersion`, `publication`. `publication` contém `id`, `capabilities`, `artifacts`, `units`, `storage`, `resources`, `artifactRelations`, `origins`, `coverage`, `uncertainties`, `premises`. | [§2](json-v1.md#2-envelope-e-versão), [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| SemanticVersion | `Publication.airVersion` Java corresponde ao componente semântico `Publication.version`; wire contém somente `airVersion:"2.0.0"` no envelope. | [§2](json-v1.md#2-envelope-e-versão) |
| Manifest / Capability | `capabilities.required=[]`, `provided=[]`; não há Capability instanciada nem declaração de perfil AIR. `minimal-entry-goback@1` é capacidade local do lower, não perfil oficial AIR a injetar. | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| Artifact | Dois artefatos, original e expandido, cada um com `id`, `logicalName`, `contentDigest=null`. Identificam fontes lógicas, não o arquivo de transporte SP/AIR. | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| Unit | Uma unidade: `id`, `containingUnit=null`, `objects=[]`, `visibleObjects=[]`, `entries` com uma Entry, `sequences` com uma Sequence, `completionPorts=[]`, `body={kind:"available"}`, `coverage`, `origin`. | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| Entry | `id`, `initialLabel` fechado no label da Sequence, `signature`, `state`, `origin`. A origem da entrada é distinta da origem do Return. | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| Signature / inventários / UnknownBound | `parameters` e `results` têm, cada um, `known=[]` e `remainder={kind:"none"}`; `origin` é o da Entry. Zero conhecido é fechado; não é assinatura desconhecida. Não há Parameter, ResultSlot, TypeRef ou operando instanciado. | [§9](json-v1.md#9-interações-assinaturas-e-premissas), [§5](json-v1.md#5-variantes-ausência-e-desconhecimento) |
| EntryState | `conditions=[]`, `uncertainties=[]`. Não há InitialCondition nem seed, objeto ou memória a inventar. Condições não fornecidas não afirmam inicialização por zero ([AIR 03 §8](../especificacao/03-memoria-e-aliases.md#8-inicialização)). | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103) |
| Sequence / Return / OperationHeader | Sequence tem `label`, `instructions=[]`, `terminator`, `origin` derivada. Terminador: `kind:"return"`, `values=[]`, `header`. Header contém `id`, `origin`, `coverage:"MODELED"`, `precision`, `uncertainties` com as quatro razões dimensionais. Sem `entryScope`, Halt, Jump ou label de saída fictício. | [§7](json-v1.md#7-operações-e-ocorrências) |
| IDs | Domínio `publication`; globais `artifact`, `unit`, `origin`, `uncertainty`; pertencentes à unidade `entry`, `label`, `operation`. Todas as referências repetem namespace completo. `CoverageItem.outputs` é heterogêneo (operation/label/entry), sem converter para texto local. | [§4](json-v1.md#4-identidades-e-fechamento) |
| Origin | Oito definições: quatro `written` (original/expandida de GOBACK e Entry) e quatro `derived` (combinações original/expandida, Sequence, Unit). `written` conserva `id,artifact,location,includes,exact`; `derived` conserva `id,inputs,rule`, sem span inventado. | [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06) |
| Location / Span / Position | As quatro origens escritas têm `line_columns(span)`, `start/end`, `lineBase="1"`, `columnBase="0"`, `columnUnit="UNICODE_SCALAR"`, `endExclusive=false`, `exact=true`, `includes=[]`. GOBACK original vai de linha `"4"`/coluna `"11"` a linha `"4"`/coluna `"16"`; demais coordenadas também são strings, preservadas. | [§3](json-v1.md#3-convenções-físicas-e-números), [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06) |
| Coverage / CoverageItem | Publication e Unit têm `inventory:"PARTIAL"`, seus próprios `scope`, `items` e referência ao gap de entradas alternativas. Cada lista `items` tem dois valores, na ordem statement → entry: `sourceKey,origin,status:"MODELED",outputs,uncertainties:[],elimination:null`. Outputs do statement são `[OperationId,LabelId]`; da entrada, `[EntryId]`. A repetição desses valores nas duas coberturas não redefine IDs. | [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06) |
| Precision / Claim | Cinco campos: `control`, `storage`, `effects`, `values`, `dependencies`. Cada Claim tem `scope,status,reasons`. Controle é `EXACT` com `reasons=[]`; as outras quatro dimensões são `UNAVAILABLE`, cada uma com sua razão. O codec não fortalece essas alegações. | [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06), [§10.4](json-v1.md#104-tokens) |
| Uncertainty / Dimension | Cinco definições no inventário global, com `id,code,dimensions,scope,reason,origin`: quatro `cobol-lower:UNPROVED_*` (STORAGE, EFFECTS, VALUES, DEPENDENCIES), mais `cobol-lower:ALTERNATE_ENTRIES_NOT_PROJECTED` (CONTROL). Header/claims/cobertura referem IDs existentes, não cópias de definições. Códigos qualificados são conservados sem inferir semântica de seu texto. | [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06), [§10.4](json-v1.md#104-tokens), [AIR 06 §4](../especificacao/06-incompletude-e-proveniencia.md#4-razões-tipadas) |
| FactScope | `publication(publication:PublicationId)` na cobertura global; `unit(unit:UnitId)` na cobertura da Unit e gap de entradas; `entities(entities:[OperationId])` nos claims/razões do Return. Não são DomainProofScope, MemoryScope nem ControlScope. | [§10.3](json-v1.md#103-origem-cobertura-e-precisão--air-06) |
| Demais inventários obrigatórios | `publication.storage=[]`, `resources=[]`, `artifactRelations=[]`, `premises=[]`. Nenhuma Premise/Assertion/DomainProofScope é instanciada ou referida pelo slice. Presença dos arrays vazios é exercitada; suas variantes não são. | [§2](json-v1.md#2-envelope-e-versão), [§9](json-v1.md#9-interações-assinaturas-e-premissas) |

**Variações de proveniência já admitidas pelo mesmo lower:** `written.location=null`
com `exact=false` quando coordenadas não sustentam Span; cadeias `IncludeFrame[]`
não vazias com `including`, `included`, `requestedName`, `site=null` quando apenas
a linha de inclusão é conhecida. Os artefatos adicionais também precisam fechar.
São formas necessárias quando presentes; a fixture acima não as exercita. A cadeia
e seus nomes não autorizam abrir arquivos nem fabricar coordenadas. Limitações e
correlações do relatório do lower não são novos campos da Publication.

`Location.offsets`, origens `contractual`/`unavailable`, valores/operandos, envelopes
conservadores e premissas com conteúdo não são produzidos por esse caminho. O
binding os cobre; seus testes precisarão de outras publicações. Em particular,
incerteza dimensional não exige acrescentar `Envelope` a `return`.

## 3. Matriz mínima de conformidade a implementar

**Todas as células “planejado” são trabalho futuro no módulo de codec JSON do
repositório `air-java`, sem implementação ou execução comprovada em 0B.** `MVP`
identifica a prioridade de evidência do primeiro E2E; não um perfil normativo nem uma lista de formas válidas
exclusivas. `Variação` exige outra entrada admitida do slice; `vazio` testa presença
e preservação do contêiner, não cobertura de seus elementos.

Encode deve preservar os fatos/ordem publicados; decode deve materializar esses
mesmos fatos sem defaults. Round-trip inclui ambos os percursos de [§12](json-v1.md#12-io-e-round-trip).
O oracle deve comparar dados, IDs, origens, scopes e razões, não apenas contar um
Return. Os casos negativos abaixo distinguem erro físico (F), violação AIR (A) e
perda/fortalecimento indevido no transporte (T); F/A seguem a seção 5 deste guia.

| Forma / regra coberta pelo binding | Encode | Decode | Round-trip | Caso negativo planejado | Exercitada no GOBACK | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Envelope / Publication / versões (§2) | planejado | planejado | planejado | F: omitir inventário ou duplicar `airVersion` dentro de Publication; versão diferente → incompatibilidade explícita | sim | MVP |
| Manifest (§10.1) | planejado | planejado | planejado | F: omitir `required`/`provided`; T: inserir perfil não declarado | vazio | MVP |
| Artifact (§10.1) | planejado | planejado | planejado | F: omitir `contentDigest`; A: redefinir ArtifactId | sim | MVP |
| Unit / BodyKnowledge.available (§10.1) | planejado | planejado | planejado | F: body como enum/string Java; A: corpo disponível sem entradas/sequências | sim | MVP |
| Entry (§10.1) | planejado | planejado | planejado | A: `initialLabel=null` com corpo disponível, inexistente ou de outra Unit | sim | MVP |
| Signature / ParameterInventory / ResultInventory (§9) | planejado | planejado | planejado | F: lista direta em vez de inventário; T: apagar restante conhecido ou substituir aridade unknown por zero | sim, ambos fechados vazios | MVP |
| UnknownBound.none (§5) | planejado | planejado | planejado | F: `null`/omissão no lugar de `{kind:"none"}` | sim | MVP |
| EntryState (§10.1) | planejado | planejado | planejado | F: omitir `conditions`/`uncertainties`; T: fabricar seed | vazio | MVP |
| Sequence (§7) | planejado | planejado | planejado | F: terminador omitido; A: Return em `instructions` ou operação comum como terminador | sim | MVP |
| Return (§7) | planejado | planejado | planejado | F: `entryScope`/`successor` extra; T: converter para Halt ou criar fallthrough | sim, `values=[]` | MVP |
| OperationHeader (§7) | planejado | planejado | planejado | F: omitir `coverage`, `precision` ou `uncertainties`; A: owner incorreto no ID | sim | MVP |
| IDs completos / unicidade / fechamento (§4) | planejado | planejado | planejado | F: ID só local; A: definição duplicada, namespace/domínio trocado, origem/razão/output pendente | sim, sete domínios além de publication | MVP |
| Origin.written (§10.3) | planejado | planejado | planejado | A: ArtifactId pendente; T: fundir origens original/expandida ou promover `exact` | sim | MVP |
| Origin.derived (§10.3) | planejado | planejado | planejado | A: `inputs=[]` ou origem pendente; F: acrescentar `location` | sim | MVP |
| Location.line_columns / Span / Position (§10.3) | planejado | planejado | planejado | F: coordenada JSON number ou string `"04"`; T: mudar base/unidade/fim | sim | MVP |
| `written.location=null` / `exact=false` (§10.3) | planejado | planejado | planejado | F: campo omitido; T: substituir por Span inventado | não na fixture; variação | MVP — proveniência |
| IncludeFrame (§10.3) | planejado | planejado | planejado | A: artefato pendente; T: reordenar cadeia ou inventar `site` | só `includes=[]`; frames em variação | MVP — proveniência |
| Coverage (§10.3) | planejado | planejado | planejado | T: PARTIAL → COMPLETE ou exclusão do gap; A: scope com referência pendente | sim, Publication e Unit | MVP |
| CoverageItem (§10.3) | planejado | planejado | planejado | F: omitir `elimination`/`uncertainties`; A: output inexistente; T: colapsar dois itens | sim | MVP |
| Precision / Claim (§10.3) | planejado | planejado | planejado | F: omitir dimensão; T: UNAVAILABLE → EXACT/NOT_APPLICABLE ou perda de razão | sim | MVP |
| Uncertainty / Dimension (§§10.3–10.4) | planejado | planejado | planejado | A: referência pendente; T: perda de código, dimensão, scope, reason ou origin | sim, cinco lacunas | MVP |
| FactScope.publication/unit/entities (§10.3) | planejado | planejado | planejado | F: trocar `entities` por um ID nu; A: domínio/referência incompatível; T: alargar scope | sim, três variantes | MVP |
| `storage/resources/artifactRelations/premises`, `objects/visibleObjects/completionPorts` (§§2, 10.1) | planejado | planejado | planejado | F: omitir arrays ou substituí-los por `null`; T: descartar conteúdo em futuras publicações | vazios | MVP — contêineres |
| Regras físicas transversais (§§3–6, 11–12) | planejado | planejado | planejado | Casos da seção 4; versão/limite não viram sucesso parcial | sim; adversariais adicionais | MVP |

O codec tem como alvo **todo o binding candidato**, incluindo o catálogo abaixo.
O recorte de evidência do primeiro E2E não autoriza apagar fatos ou aceitar outra
gramática. Suporte ainda não implementado deverá falhar explicitamente, com limite
ou incompatibilidade identificado, nunca produzir uma Publication mutilada. Uma
forma válida fora da cobertura implementada não vira `INVALID_IR` por esse motivo.

| Binding cobre também | Exercitada no testemunho | Evidência de encode/decode/round-trip/negativos em 0B | Prioridade |
| --- | --- | --- | --- |
| Capability com conteúdo, outras versões declaradas/negociação (§§1, 10.1, 11) | não; Manifest vazio | não demonstrada; ampliar casos além da fixture | DEFERRED — cobertura ampliada |
| Parameter/ResultSlot, modos, bindings, restantes unknown, TypeRef/Type (§§5, 9) | não; só inventários fechados vazios | não demonstrada | DEFERRED — catálogo fora do slice |
| OperandHeader, OperandId e owners, Expression/Place/literais (§§3–4, 7–8) | não; Return sem valores | não demonstrada | DEFERRED — catálogo fora do slice |
| Outras operações core, invocações, targets, ContractRef, efeitos/outcomes/envelopes e pontos (§§7–10.2) | não | não demonstrada | DEFERRED — catálogo fora do slice |
| Object, Storage/Binding/Codec, InitialCondition/Value, Resource e ArtifactRelation com conteúdo (§10.1) | não | não demonstrada | DEFERRED — catálogo fora do slice |
| Premise/Assertion, DomainSubject/DomainProofScope (§9), demais scopes/bounds (§10.2) | não | não demonstrada | DEFERRED — catálogo fora do slice |
| BodyKnowledge.unavailable, Origin.contractual/unavailable, Location.offsets, Elimination (§§10.1, 10.3) | não | não demonstrada | DEFERRED — catálogo fora do slice |
| `memory.regions@1`, `control.local@1`, `control.indirect@1` (§§1, 7–10) | não | não demonstrada | DEFERRED — catálogo fora do slice |

Adiamento de evidência ampla não dispensa regras físicas: Base64, inteiros grandes,
Unicode e unknown precisam de casos dirigidos ao serem implementados, ainda que
o GOBACK não forneça valores para todos eles. Não se anuncia seção 13 cumprida com
base na matriz mínima.

## 4. Índice operacional das regras físicas

As referências abaixo apontam às regras da **candidata DRAFT pinada**, subordinadas
aos normativos AIR. São lembretes de implementação e casos planejados, não uma nova
definição de transporte. “Canônico” qualifica a escrita; entrada JSON válida pode
ter outra ordem de propriedades, whitespace e escapes equivalentes permitidos.

| Regra | Obrigação do draft e verificação dirigida | Autoridade |
| --- | --- | --- |
| UTF-8, BOM e documento único | Um objeto JSON em UTF-8, sem BOM; rejeitar UTF-8 inválido, segundo documento e lixo final. Só whitespace JSON pode seguir o objeto. Exercitar também raiz não objeto. | [§3](json-v1.md#3-convenções-físicas-e-números) |
| Strings Unicode | Rejeitar surrogates isolados, inclusive escapes; conservar escalares, caixa, padding, texto vazio e Unicode suplementar. Não aplicar normalização Unicode/ambiental. | [§3](json-v1.md#3-convenções-físicas-e-números) |
| Propriedades duplicadas | Rejeitar em qualquer profundidade antes que um mapa possa sobrescrever valor. Incluir chave equivalente por escape, como `binding` e `\u0062inding`. Duplicação de propriedade difere de repetição de ID por referência. | [§3](json-v1.md#3-convenções-físicas-e-números), [§11](json-v1.md#11-validação-e-campos-desconhecidos) |
| Envelope e versões exatas | Exigir `binding="analysis-ir-json"`, `bindingVersion="1.0.0"`, `airVersion="2.0.0"`; não usar versão Maven nem só comparar major. Versão divergente demanda incompatibilidade explícita, sem leitura por semelhança. `Publication.version` é codificada uma única vez. | [§2](json-v1.md#2-envelope-e-versão), [§11](json-v1.md#11-validação-e-campos-desconhecidos) |
| Inteiros / naturais | Todos são strings decimais canônicas, inclusive coordenadas, posições, bases e escalas. Rejeitar número JSON, `-0`, `+1`, `01`, fração e expoente; Natural não admite negativo. Testar acima de 2^53 e dos limites de tipos primitivos. Limite operacional não autoriza truncar. | [§3](json-v1.md#3-convenções-físicas-e-números) |
| Decimal | Conservar coeficiente e escala; igualdade numérica não autoriza reescrever a representação transportada no round-trip. Sem arredondamento do runtime. Não há decimal instanciado no GOBACK. | [§3](json-v1.md#3-convenções-físicas-e-números), [§12](json-v1.md#12-io-e-round-trip) |
| Base64 canônico | Alfabeto padrão `A–Z a–z 0–9 + /`, grupos de quatro, padding necessário, bits de padding zero, sem whitespace; vazio = zero octetos. Rejeitar forma URL-safe, padding inválido/não canônico e arrays de números como bytes. Casos dirigidos `00/FF`; nenhum byte literal no GOBACK. | [§3](json-v1.md#3-convenções-físicas-e-números) |
| Discriminadores / campos | Soma usa `kind` explícito; não inferir pela classe Java ou presença de campo. Todos os campos catalogados são obrigatórios. Rejeitar campo, kind ou token fechado desconhecido; sem payload livre ou desserialização por nome de classe. | [§5](json-v1.md#5-variantes-ausência-e-desconhecimento), [§11](json-v1.md#11-validação-e-campos-desconhecidos) |
| `null`, ausência, unknown | Somente `?` permite campo presente com `null` para ausência real. Omitido não vira `null`, `[]`, zero, `false` ou `none`. Unknown usa variante/lacuna explícita; por exemplo `unknown_type` fecha em `TYPE_UNKNOWN`. Testar `null` no lugar de remainder unknown e perda de referência à lacuna. | [§5](json-v1.md#5-variantes-ausência-e-desconhecimento), [§9](json-v1.md#9-interações-assinaturas-e-premissas) |
| IDs / namespaces | Emitir domínio e todos os owners em definições e referências, sem joins por `localId`/nomes. `publication` e `unit` dentro do ID são textos locais dos owners. Region/cell são espécies de StorageId; não há domínio contract. IDs longos do lower são opacos, não hashes a recalcular. | [§4](json-v1.md#4-identidades-e-fechamento) |
| Unicidade / fechamento | Uma definição por ID completo; referências repetidas são legítimas. Fechar IDs antes de disponibilizar a Publication; testar referências cruzadas e duplicadas. Unicidade de posições/conjuntos segue o modelo, não deduplicação geral de arrays. | [§4](json-v1.md#4-identidades-e-fechamento), [§6](json-v1.md#6-ordenação-e-canonical-json), [§12](json-v1.md#12-io-e-round-trip), [I-01–I-05](../conformidade/01-invariantes.md#1-classes-de-verificação) |
| Ordem de arrays | Preservar todos os arrays na ordem transportada. Ordem de operações/posições/cadeias tem papel semântico pertinente; ordem de inventários/sequências não cria execução ou precedência. Testar array não vazio e permutação que não deve ser ordenada pelo encoder. | [§6](json-v1.md#6-ordenação-e-canonical-json) |
| Canonical JSON | Propriedades em ordem lexicográfica por escalares Unicode; sem indentação, sem newline final, saída UTF-8. Mesmos fatos **e mesma ordem física** geram mesmos bytes. Não é promessa de bytes iguais após renomear IDs, permutar inventários ou mudar decomposição. | [§6](json-v1.md#6-ordenação-e-canonical-json) |
| Escaping canônico | Escapar só aspas, barra invertida e U+0000..U+001F; usar `\b`, `\f`, `\n`, `\r`, `\t` onde cabíveis e `\u00xx` minúsculo nos outros controles. Não escapar `/` nem demais escalares. Testar saída literal suplementar e recanonicalização de escapes válidos. | [§6](json-v1.md#6-ordenação-e-canonical-json) |
| Metadados / determinismo | Encoder não acrescenta timestamp, IDs aleatórios nem propriedades de processo. Conserva IDs/metadados publicados admitidos no catálogo. A data deste snapshot é documentação, não campo wire. Rodadas repetidas não podem mudar bytes por relógio/ambiente. | [§6](json-v1.md#6-ordenação-e-canonical-json) |
| Capabilities e tokens extensíveis | Capability conserva nome e versão declarada: `"1"` para extensão padronizada @1, `"2"` para perfil @2. Não extrair major de versão arbitrária. Códigos qualificados só onde AIR permite; sem suporte preciso, usar fallback existente permitido ou incompatibilidade, nunca inventá-lo. | [§10.1](json-v1.md#101-publicação-unidades-memória-e-entrada--air-0103), [§11](json-v1.md#11-validação-e-campos-desconhecidos), [AIR 09 §6](../especificacao/09-extensibilidade-e-compatibilidade.md#6-negociação) |
| Limites de implementação | Tempo, memória, cardinalidade, tamanho de número/ID/profundidade geram falha explícita no escopo afetado. Sem overflow, truncamento, prefixo válido ou regra AIR nova; não expor Publication parcialmente decodificada. O draft não fixa tetos nem API Java de limites. | [§3](json-v1.md#3-convenções-físicas-e-números), [§12](json-v1.md#12-io-e-round-trip) |
| Round-trip e indivisibilidade | `decode(encode(P))` conserva fatos, evidências, IDs e restantes; `encode(decode(J))` conserva dados/arrays e gera forma canônica. Comparar versão, cinco lacunas, scopes, origens, cobertura e assinatura, além do controle. Writer pode substituir arquivo após escrita completa; essa política não redefine AIR. | [§12](json-v1.md#12-io-e-round-trip), [§13](json-v1.md#13-oráculos-para-promoção-e-codecs) |

## 5. Fronteira de erros e mapeamentos Java

| Classe | Tratamento previsto pela autoridade |
| --- | --- |
| Física / transporte | Bytes, léxico, propriedades duplicadas, campos/variantes malformados: `INPUT_ERROR` (§11). Não perder duplicatas antes de validar. |
| Incompatibilidade de versão | Diagnóstico explícito de versão não suportada (§§2, 11). O draft não fixa um nome de enum específico para version mismatch; a API futura deve distingui-lo, sem negociar silenciosamente. |
| AIR inválida | JSON bem formado pode ter ID pendente, owner incorreto, posição/assinatura contraditória ou outra violação AIR: `INVALID_IR`, com regra/site (§11; I-01–I-08 etc.). Construtor Java rejeitar algo não basta para classificar toda falha como erro físico. |
| Capacidade não suportada | `UNSUPPORTED_CAPABILITY`, ou fallback já publicado quando permitido (§11). Não ignorar campo/operação semântico e não tratar falta de implementação como defeito do programa. |
| Validação inconclusiva | `INCOMPLETE_VALIDATION` registra obrigações/limites; não é aprovação integral nem licença para reparar fatos (§11). Forma válida não prova verdade do lowering, pureza ou premissas. |
| Limite operacional de transporte | Falha explícita, sem Publication parcial (§§3, 12). Seus tipos/limites concretos cabem à futura API do codec; não se confundem com `PARTIAL` semântico já publicado. |
| Falha de preservação (T na matriz) | Perder uma origem/razão, elevar precisão ou alterar Return é defeito do codec, mesmo quando a saída alterada pareça estruturalmente válida. Detectar pela comparação com o oracle independente, não só pelo Validator. |

No Java consultado, os mapeamentos necessários já têm fatos representáveis, mas
não correspondem automaticamente ao wire:

| Shape Java | Mapeamento explícito para o draft |
| --- | --- |
| `Publication.airVersion: SemanticVersion(BigInteger,BigInteger,BigInteger)` | String semver no envelope; não objeto `major/minor/patch` dentro de Publication. |
| `Unit.body` + `bodyUnavailable: Optional<UncertaintyId>` | Uma soma `BodyKnowledge`: `available` no slice; `unavailable(uncertainty)` se pertinente. Não publicar `bodyUnavailable` separado. |
| IDs com proprietários tipados aninhados | Domínio lexical do binding e owners textuais completos (§4); não serializar recursivamente todos os records nem `PublicationId.publication()` autorreferente. |
| `Optional` e singleton `NoRemainder.INSTANCE` | Campo `?` com `null` quando permitido; restante conhecido vazio como `{kind:"none"}`. Ausência Java não é unknown AIR. |
| `Operations.Return` / `Origins.*` / `Scopes.*` | Discriminadores `return`, `written`, `derived`, `line_columns`, `publication`, `unit`, `entities` nas formas catalogadas; enums de status seguem tokens de §10.4, sem nomes de classes. |
| `BigInteger`, listas e `Evidence.*` | Strings numéricas canônicas, arrays com ordem preservada e referências por ID onde prescritas. Shared object em memória não vira ID novo nem elimina valores repetidos de CoverageItem. |
| Relatório de validação/limitações/correlações do lower | Evidência externa ao wire AIR, salvo fatos já materializados no catálogo. Não criar campo genérico `evidence`, `metadata`, `contracts` ou campos CFG. |

O codec compartilhado servirá `cobol-lower` e `analysis-cfg`. AIR JSON transporta
AIR; CFG JSON é outro contrato, de propriedade de `analysis-cfg` ([AIR 08 §§1–2](../especificacao/08-contrato-de-consumidores.md#1-regras-comuns)).
A comparação futura entre consumo em memória e por arquivo usa a mesma Publication,
perfis e premissas; não exige implementar CFG neste repositório.

## 6. Decisões e itens DEFERRED

| Item | Status / razão e condição para retomar |
| --- | --- |
| Segunda implementação independente | **DEFERRED** — hardening futuro; necessária antes de alegar interoperabilidade independente nos termos de §13. |
| Cross-language interoperability / conformance | **DEFERRED** — não é necessária para transportar AIR entre os aplicativos Java atuais; exige evidência específica futura. |
| Cross-codec qualification | **DEFERRED** — lower e CFG reutilizarão o mesmo codec Java; dois callers não contam como duas implementações independentes. |
| Promoção normativa completa | **DEFERRED** — DRAFT permanece; revisão explícita e evidência de promoção continuam exigidas. |
| Full section-13 qualification | **DEFERRED** — a matriz mínima não demonstra o catálogo inteiro, contracasos O-82–O-91, todos os perfis ou interoperabilidade. Requisitos não foram removidos de §13. |
| Claim de interoperabilidade universal | **DEFERRED / não autorizado no MVP** — não há prova de compatibilidade com qualquer linguagem, implementação ou versão. |
| Cobertura completa de testes do catálogo fora do slice | **DEFERRED** conforme seção 3; ampliar por novas formas e capacidades, conservando como alvo o binding inteiro pinado. |
| Extensões futuras do binding | **DEFERRED** — fora de 0B; exigem proposta e política de versão/negociação próprias. |

Esses itens continuam válidos como hardening futuro e **não bloqueiam o primeiro
E2E**. Um codec Java compartilhado, implementado e validado contra este pin, será
suficiente para o ecossistema atual no MVP. Isso é a decisão de arquitetura e o
alvo da próxima implementação; não uma alegação de codec já entregue.

Depois da implementação e da evidência do E2E, o claim poderá se limitar a transporte
AIR versionado e pinado entre os aplicativos atuais pelo codec compartilhado.
Não se poderá afirmar “AIR JSON 1.0.0 plenamente qualificado e independentemente
interoperável”, nem conformidade integral de perfil só porque GOBACK passou.

## 7. Findings, validação documental e handoff

Não foi encontrada inconsistência objetiva do draft que bloqueie a representação
da Publication GOBACK identificada. Não foi necessária mudança normativa.
Foram registrados como pontos de implementação: a parcialidade global apesar do
controle local exato; arrays vazios obrigatórios versus variantes não exercitadas;
origens original/expandida e includes condicionais; e os mapeamentos Java da seção 5.
O nome local do roadmap e a diferença entre checkout Java e pin do lower estão
documentados nas seções 1–2, sem atualizar dependências ou outros repositórios.

**Este checkpoint organiza e fixa a base de implementação do draft atual; não
promove nem altera a semântica do binding.** Não contém codec, Java, módulo Maven,
encoder/decoder, schema paralelo, CFG, CLI ou testes cross-language.

O repositório no baseline contém somente documentação: não há build, suite
executável de codec, script de validação ou workflow de CI versionado. A revisão
documental de 0B verifica diff/whitespace, links e âncoras locais, referências a
invariantes/oráculos/exemplos, estrutura de tabelas e parsing dos blocos JSON
literais, além da correspondência dos campos desta matriz com a candidata pinada.
Esses checks não provam encode/decode, round-trip, verdade semântica ou a seção 13.
Comandos e resultados executados são registrados no PR de entrega.

O próximo agente em `air-java` deve usar a seção 1 como pin, §§2–3 como inventário
de formas/evidência inicial e §§4–5 como índice de regras/mapeamentos. Antes de
afirmar transporte funcional, deverá produzir testes e resultados no codec,
preservando todos os fatos da Publication e os limites declarados. Esse trabalho
posterior não foi iniciado por 0B. A entrega deste checkpoint termina no PR para
review humano, sem merge automático.
