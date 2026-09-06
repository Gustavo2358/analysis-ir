# 01 — Modelo abstrato e identidades

**Analysis IR 2.0.0 — Normativo**

## 1. Publicação

Uma `Publication` é um conjunto fechado e imutável de fatos de IR. Ela DEVE conter os componentes abaixo, mesmo que algum inventário esteja indisponível de forma explícita.

| Componente | Cardinalidade | Significado |
| --- | --- | --- |
| `publicationId` | 1 | Namespace da publicação; não é nome do programa |
| `version` | 1 | Versão semântica desta especificação |
| `capabilities` | 1 | Capacidades requeridas e disponibilizadas, com versão |
| `artifacts` | 0..N | Identidades lógicas dos artefatos de origem ou recursos estruturais |
| `units` | 0..N | Unidades executáveis e seus inventários |
| `storage` | 0..N | Células, regiões e suas propriedades declarativas |
| `resources` | 0..N | Identidades internas e nomes externos tipados |
| `artifactRelations` | 0..N | Relações estruturais, independentes de execução |
| `origins` | 0..N | Âncoras de proveniência e derivações |
| `coverage` | 1 | Inventário de cobertura, incluindo escopo e disponibilidade |
| `uncertainties` | 0..N | Lacunas referenciadas por fatos ou escopos |
| `premises` | 0..N | Premissas semânticas identificadas e rastreáveis, inclusive `sameDomain` com sujeitos/escopo conforme [02](02-tipos-valores-e-operandos.md#14-escopo-de-provas-de-domínio) e separação entre bases conforme [03](03-memoria-e-aliases.md#31-evidência-declarativa-de-separação) |

Uma publicação vazia é permitida apenas se seu escopo e a disponibilidade do inventário distinguirem “zero conhecido” de “não analisado”.

O inventário é fechado: **não existe `Publication.contracts`, entidade `Contract` ou domínio `ContractId` no núcleo desta edição**. O conteúdo contratual usado por uma interação é materializado no próprio `invoke` e nas premissas referidas por seus sujeitos/sites, conforme §9 e [04, §7.4](04-operacoes.md#74-materialização-de-contratos-e-assinaturas). Isso é independente de uma implementação internar valores ou manter índices privados.

## 2. Unidades e entradas

Uma `Unit` possui identidade, origem, contenção opcional, declarações visíveis, uma coleção de entradas e uma coleção de sequências. Cada `Entry` possui identidade, assinatura normalizada de parâmetros e resultados e label inicial quando seu corpo está disponível, conforme as condições abaixo.

A assinatura especifica parâmetros em ordem, modo de passagem, `TypeRef` e, para valores recebidos, o objeto inicializado na entrada. Resultados têm `TypeRef` e ordem declarados. Uma posição de tipo desconhecido usa `unknown_type(u)` sem apagar posição, modo ou objeto conhecidos. Isso é distinto de desconhecer a quantidade de posições ou a assinatura inteira. Assinatura desconhecida é admissível somente com incerteza correspondente; o consumidor NÃO DEVE tratá-la como assinatura vazia. As condições de transmissão são definidas em [04 — Operações](04-operacoes.md).

Uma unidade com corpo disponível DEVE possuir pelo menos uma entrada. Cada entrada de corpo disponível aponta para uma sequência existente. Numa unidade declarada com corpo indisponível e lacuna correspondente, a declaração de entrada pode conservar sua assinatura sem label inicial; ausência de label não é permitida para corpo disponível, e qualquer label publicado deve fechar sobre sequência existente. Múltiplas entradas NÃO DEVEM ser fundidas sem preservar quais inicializações e parâmetros valem para cada uma. Conteúdo não alcançável de uma entrada pode ser alcançável de outra.

Unidades contidas não herdam visibilidade por inferência. Uma captura ou referência a estado de outra unidade DEVE designar explicitamente um objeto visível e seu armazenamento. O produtor resolve visibilidade; o consumidor não repete lookup nominal.

## 3. Sequências

Uma `Sequence` é identificada por um `LabelId` e contém uma lista ordenada de zero ou mais operações comuns, seguida de exatamente um terminador. Um corpo disponível contém pelo menos uma sequência. A entrada no label inicia sua primeira operação, inclusive quando ela é apenas o terminador.

A ordem de operações dentro da sequência é semântica. A ordem das sequências na publicação NÃO é ordem de execução. Não existe fallthrough implícito entre sequências. Um label não pode entrar no meio de uma sequência: quando necessário, ela deve ser dividida sem alterar as operações observadas.

Uma sequência não precisa corresponder a uma construção-fonte ou a um bloco básico máximo. Dividir uma sequência em duas ligadas por `jump` é representação equivalente quando preserva operações, pontos observáveis e proveniência. O CFG pode conservar ou agrupar tais sequências sob um mapeamento explícito.

## 4. Identidades

Os domínios obrigatórios são `ArtifactId`, `ArtifactRelationId`, `UnitId`, `EntryId`, `LabelId`, `OperationId`, `OperandId`, `ObjectId`, `StorageId`, `ResourceId`, `OriginId`, `UncertaintyId` e `PremiseId`. `ArtifactRelationId` explicita a identidade da relação já exigida em §8; pertence à publicação. `publicationId` identifica o namespace exterior. `CompletionPortId` é acrescentado por `control.local@1`. Extensões podem acrescentar domínios, não reinterpretar os existentes.

Cada identidade tem significado no namespace completo da publicação e de seu proprietário. Um número ou texto local isolado NÃO DEVE ser usado como chave global. Identidades de domínios diferentes NÃO são intercambiáveis, mesmo que tenham a mesma grafia.

Artefatos, relações, unidades, armazenamento, recursos, origens, lacunas e premissas pertencem ao namespace da publicação. Entradas, labels, operações, objetos e portas pertencem a uma unidade. Ocorrências de operandos pertencem à operação que as avalia ou à entrada cuja condição inicial as declara. Referências a essas ocorrências em envelopes não criam novas ocorrências. A unidade/duração de um armazenamento é uma propriedade declarativa adicional, não uma abreviação de sua identidade.

Uma operação pertence a exatamente uma sequência e unidade. Cada ocorrência de operando possui identidade própria, distinta da identidade do objeto que pode referenciar. Literais iguais podem ter identidades de ocorrência diferentes. Uma operação reutilizada em dois caminhos por referência ao mesmo label continua uma única operação; duas operações geradas separadamente exigem identidades distintas.

Identidades IR são próprias. O produtor pode manter correlações opacas com sua entrada na proveniência, mas o consumidor NÃO DEVE exigir IDs, classes ou formatos de identidade de outro produto.

## 5. Pontos de programa

Um `ProgramPoint` identifica a posição semântica relativamente a uma operação:

```text
before(operation)
after(operation, outcome)
entry(unit, entryId)
exit(unit, outcome)
```

Para atribuições e operações comuns, o resultado local é `normal`. Para interações, `outcome` identifica retorno normal, exceção ou outro resultado declarado. Um valor usado como alvo ou argumento é observado em `before(invoke)`, não após seus possíveis efeitos.

Pontos não são números de linha, offsets de arquivo ou posições de travessia. Uma consulta que mistura resultados anteriores e posteriores à mesma operação é inválida. Identidade de ponto não promete que ele seja alcançável.

## 6. Declarações de objetos

Um `Object` é uma entidade nominal normalizada. Possui `ObjectId`, conhecimento de tipo de valor por `TypeRef`, origem, duração/visibilidade necessárias e uma associação declarativa de armazenamento, conforme [03 — Memória](03-memoria-e-aliases.md). `TypeRef` segue [02 — Tipos](02-tipos-valores-e-operandos.md): `known(T)` ou `unknown_type(u)` com lacuna `TYPE_UNKNOWN` explícita.

Nome de exibição é opcional e não participa de joins. Dois objetos podem nomear a mesma célula ou vistas sobrepostas de uma região. Identidade nominal não prova independência física. Objetos de tipo desconhecido continuam presentes; operações sobre eles devem usar semântica compatível com o conhecimento disponível.

## 7. Referências internas e externas

Referências internas DEVEM fechar sobre a publicação. Uma referência externa DEVE ser um `ResourceRef` explícito, nunca um ID interno inexistente. Uma unidade referida mas sem corpo pode ser representada como recurso externo ou unidade declarada com corpo indisponível e contrato de interação.

Um `ResourceRef` contém categoria, namespace de nomes, nome literal ou expressão que o calcula, política de interpretação do nome e origem. As categorias padrão são `program`, `service`, `file`, `table`, `schema` e `artifact`. Categorias adicionais exigem identificador qualificado de extensão.

Um nome externo observado NÃO é identidade confirmada de um artefato executável. Resolução contra catálogo é produto separado. Um target interno usa `EntryId`; um target externo usa nome/namespace; um target dinâmico usa expressão tipada. Eles não devem ser confundidos.

`ResourceId` identifica uma declaração do inventário `resources`, com descrição e origem; não é uma quarta forma de target executável. Um uso conserva a forma interna, literal ou calculada e a origem daquele uso. Reutilizar uma descrição literal não cria identidade de entrada nem resolve catálogo. Um nome calculado conserva sua ocorrência na operação que o avalia; não é um valor capturado por uma declaração de recurso. Relações sem ponto de execução usam destino `ArtifactId` ou descrição externa literal, não expressão que precisaria de estado de execução.

A política de nome é `exact`, uma regra de extensão identificada/versionada e negociada, ou desconhecimento com lacuna. `exact` conserva os escalares do nome. Uma autoridade/versão sem regra não define normalização. Regras já executadas pelo produtor podem ser operações explícitas antes do uso; regras especializadas exigem manifesto conforme [09](09-extensibilidade-e-compatibilidade.md). Nenhuma política permite recuperar fatos ausentes por lookup.

## 8. Relações de artefato

`ArtifactRelation` possui identidade, origem, artefato de origem, destino interno ou recurso externo, espécie e cobertura. Espécies padrão: `includes`, `uses_schema` e `declares_resource`. A direção é do artefato que usa ou declara para o referenciado.

Essas relações não possuem ponto de execução e NÃO DEVEM ser convertidas em invocações fictícias. Podem alimentar consumidores de dependências estruturais diretamente. Ausência de relações só prova inexistência quando o inventário pertinente está completo.

## 9. Fechamento e revisões

A publicação DEVE ser indivisível do ponto de vista de seus consumidores. Os fatos indispensáveis de um contrato externo devem estar materializados em tipos IR ou no limite conservador correspondente. `ContractRef` é um valor de rastreabilidade composto por autoridade, versão da autoridade e evidência não vazia de `OriginId` fechados sobre a publicação. Não é identidade de entidade, endereço, nome de classe nem chave de um inventário externo. A versão da autoridade identifica a revisão exata da evidência; não precisa seguir o versionamento da AIR.

`ContractRef` não contém uma semântica a buscar: a assinatura normalizada, limites de efeitos, outcomes e premissas aplicáveis constituem o conteúdo disponível, com as regras de [04, §7.4](04-operacoes.md#74-materialização-de-contratos-e-assinaturas). Sua ausência usa `unknown(u)` com `CONTRACT_UNKNOWN`; não apaga fatos independentes estabelecidos pelo produtor ou corpo. Remover o acesso à autoridade após publicar não muda nenhuma observação. Conhecimento adicional obtido posteriormente deve originar publicação/revisão ou produto derivado explicitamente correlacionado. Fatos de revisões diferentes não podem ser combinados apenas porque seus IDs locais coincidem. Uma análise derivada identifica `publicationId`, versão, perfis e premissas utilizadas.

Para entradas e contexto semântico equivalentes, o produtor DEVE assegurar representação semanticamente determinística. Uma codificação pode estabelecer determinismo byte a byte em contrato separado. A V2 não exige estabilidade longitudinal de IDs após edições, normalizações diferentes ou mudança de versão.
