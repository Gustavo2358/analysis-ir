# 06 — Cobertura, incompletude e proveniência

**Analysis IR 2.0.0 — Normativo**

## 1. Inventário de cobertura

Cada unidade declara `inventory=complete|partial|unavailable` e o domínio ao qual essa alegação se refere. Cada item significativo conhecido da entrada é associado a operações, declarações, relações de artefato ou lacunas. A associação admite uma entrada para várias operações e várias entradas para uma operação, mas deve preservar explicação.

Classificações padrão de item são `MODELED`, `ABSTRACTED`, `UNSUPPORTED` e `INPUT_MISSING`. `MODELED` não significa que todas as análises têm precisão exata. `ABSTRACTED` significa representação conservadora intencional. `UNSUPPORTED` mantém construção observada sem interpretação específica. `INPUT_MISSING` registra informação necessária indisponível.

Trecho não executável pode ser representado por declaração/relação, sem operação fictícia. Trecho executável com efeito conhecido nulo pode mapear a `nop` ou ser contabilizado como eliminação semanticamente justificada. Trecho desconhecido nunca desaparece.

## 2. Precisão dimensional

As dimensões são `control`, `storage`, `effects`, `values` e `dependencies`. Cada alegação tem escopo, estado e razões.

| Estado | Significado |
| --- | --- |
| `EXACT` | A regra local da dimensão está completamente especificada para o domínio declarado |
| `CONSERVATIVE` | Há sobreaproximação explícita dentro de limite fechado conhecido |
| `OPEN` | Existe restante não enumerado ou limite aberto |
| `UNAVAILABLE` | Não há produto interpretável para aquela dimensão no escopo |
| `NOT_APPLICABLE` | A dimensão não se aplica ao fato |

`EXACT` em controle de `branch` significa regra de dois destinos conhecida, não verdade conhecida do predicado. `EXACT` em efeitos de `assign` significa semântica da atualização conhecida, não destino físico necessariamente resolvido; nesse caso storage pode permanecer aberto.

Nenhuma dimensão é elevada por outra. Um literal conhecido não resolve automaticamente o recurso externo ao qual ele dará nome. Uma origem exata não torna um valor exato. Um CFG completo estruturalmente não torna os efeitos completos.

O conhecimento de domínio é representado por `TypeRef`, não por um novo estado global de precisão. `unknown_type` limita a interpretação de valores, mas não abre automaticamente storage, efeitos, controle ou dependências. Uma alegação local de leitura exata pode conservar domínio desconhecido; ela não alega valor ou tipo concreto conhecido.

Uma lacuna de identidade concreta do domínio pode coexistir com `sameDomain` comprovado. A premissa de igualdade de domínios é conhecimento positivo com sujeitos, autoridade e escopo; não substitui `TYPE_UNKNOWN` nem é inferida da razão da lacuna. Tipo concreto desconhecido não obriga a perder uma relação de cópia estabelecida por `assign` ou transmissão sem conversão. Igualdade de domínio, isoladamente, não afirma igualdade de valores.

Uma síntese global não deve ser mais forte que os fatos relevantes para o mesmo escopo e dimensão. `NOT_APPLICABLE` é excluído da combinação, não usado para esconder lacuna. Alegações locais podem ser melhores que a síntese global quando sua independência estiver demonstrada.

## 3. Envelopes conservadores

Toda operação opaca e toda extensão deve permitir a interpretação abaixo sem conhecer seu payload específico.

### 3.1 Memória

`MemoryEnvelope` contém leituras conhecidas, limite de outras leituras, escritas conhecidas, limite de outras escritas e, quando comprovadas, sobrescritas completas obrigatórias. Cada limite é `none` ou um `MemoryScope` explícito.

Um limite de escrita superior indica `may_write`, não `must_write`. Um campo de escrita obrigatória só pode ser usado com prova/contrato. O desconhecimento absoluto usa o maior escopo potencialmente visível, inclusive externo. Uma operação sem efeitos só pode ter limites vazios com justificativa semântica.

### 3.2 Controle

`ControlEnvelope` contém alternativas conhecidas de destino/saída e `remainder=none|ControlScope`. Alternativas conhecidas são possibilidades sustentadas, não destinos obrigatórios. Um restante aberto pode incluir os mesmos destinos; a redundância não torna o conjunto fechado.

Uma continuação normal conhecida deve constar como tal. Ausência de arestas conhecidas com restante aberto não é término. Comportamento sem próximo ponto, como divergência, deve estar explícito.

### 3.3 Dependências

`DependencyEnvelope` contém usos de recurso conhecidos e limite de outros usos. Um uso conhecido identifica categoria, ação, namespace, target literal ou operando calculado, ponto de observação e origem.

`none` declara que nenhuma outra dependência desse escopo pode ser introduzida. `unknown(categoryScope)` informa que outros recursos das categorias indicadas podem ser usados. `any_resource` é o limite máximo. Uma operação com memória e controle conhecidos ainda pode ter dependências desconhecidas.

O envelope não é o grafo final de dependências. Ele descreve a superfície de interação que os consumers devem respeitar.

## 4. Razões tipadas

Cada lacuna possui identidade, código estável, domínio afetado, âncora/escopo, motivo e proveniência disponível. Mensagem humana não participa de decisão semântica.

Códigos mínimos: `INPUT_MISSING`, `REFERENCE_UNRESOLVED`, `REFERENCE_AMBIGUOUS`, `TYPE_UNKNOWN`, `STORAGE_UNKNOWN`, `ALIAS_UNKNOWN`, `CODEC_UNKNOWN`, `PREDICATE_UNKNOWN`, `CONTROL_UNKNOWN`, `EFFECT_UNKNOWN`, `RESOURCE_TARGET_UNKNOWN`, `CONTRACT_UNKNOWN`, `EXTENSION_UNSUPPORTED`, `ANALYSIS_LIMIT`, `EXTERNAL_VALUE_UNKNOWN`, `ENTRY_STATE_UNKNOWN`, `UNINITIALIZED_READ` e `SOURCE_SEMANTICS_UNAVAILABLE`.

`EXTERNAL_VALUE_UNKNOWN` e `UNINITIALIZED_READ` não significam input-fonte ausente: um programa completamente representado pode receber valores externos ou ler estado não inicializado. `INPUT_MISSING` identifica falta de um artefato/fato de entrada necessário à produção; esses estados não devem ser confundidos nas métricas de cobertura.

`TYPE_UNKNOWN` ancora `unknown_type(u)` e significa domínio de valor não estabelecido. O código isolado em diagnóstico não substitui o `TypeRef` normativo. As dimensões abaixo são independentes e suas lacunas aplicáveis DEVEM coexistir:

| Situação | Representação e limite |
| --- | --- |
| Valor desconhecido, domínio conhecido | `unknown(known(T),...,reason)`; razão de valor |
| Domínio desconhecido | `unknown_type(u)` com `TYPE_UNKNOWN`; pode acompanhar `read` ou `unknown` |
| Associação física desconhecida | Binding `unknown(scope,reason)` com `STORAGE_UNKNOWN`; conserva tipo conhecido se disponível |
| Binding nominal não resolvido/ambíguo | `choice`/referência aberta com `REFERENCE_UNRESOLVED` ou `REFERENCE_AMBIGUOUS`; preserva candidatos e seus tipos |
| Domínio de extensão identificado | `known(opaque_type(id,version))`; falta de interpretação usa `EXTENSION_UNSUPPORTED`, não `TYPE_UNKNOWN` |

Em `unknown(unknown_type(u),...,reason)`, `u` explica a lacuna de tipo e `reason` explica a de valor. Origens e causas podem estar relacionadas, mas nenhum desses fatos substitui o outro. Em `read(p)`, preservam-se a lacuna de tipo de `p` e as razões aplicáveis ao conteúdo/armazenamento; não se inventa desconhecimento de binding porque o domínio não foi estabelecido.

Implementações podem acrescentar códigos qualificados, mas não redefinir os existentes. Um desconhecimento de target não deve ser registrado como falha de binding do objeto que contém o nome.

## 5. Proveniência

Uma origem inclui identidade de artefato lógico, localização quando disponível, exatidão e cadeia de transformação/inclusão relevante. Localizações especificam unidade de medida: linhas/colunas devem declarar base e convenção; offsets físicos usam octetos ou outra unidade explicitamente identificada. Não se misturam offsets de texto e bytes de armazenamento.

A origem de um fato pode ser `WRITTEN`, `DERIVED`, `CONTRACT` ou `UNAVAILABLE`. `DERIVED` referencia uma ou mais origens e a regra de derivação. Não recebe span-fonte inventado. Um identificador auxiliar pode ser derivado de uma expressão sem aparecer como declaração escrita.

Cada operação, operando, declaração, relação de artefato e lacuna deve ter origem ou `UNAVAILABLE` explícito. Origem aproximada é permitida, mas sua exatidão não pode ser promovida por agregação. O conteúdo-fonte completo não precisa acompanhar a IR.

Metadados de diagnóstico podem conservar grafias e nomes da construção de origem. Consumidores não devem analisá-los para obter semântica. Remover metadados de exibição não pode alterar resultados semânticos; remover origem reduz explicabilidade e pode violar o perfil declarado.

## 6. Fatos conhecidos e restante desconhecido

Conjuntos de candidatos possuem representação conceitual `(enumerated, unknownRemainder)`. O restante é uma possibilidade de informação não enumerada, não um elemento literal do conjunto.

Exemplo: um ramo escreve um nome literal e outro recebe um nome externo. A consulta pode conservar o literal e marcar restante aberto. Porém, se depois ambos os caminhos sofrem sobrescrita completa obrigatória por valor desconhecido, aquele literal deixa de ser valor corrente sustentado. Ele permanece apenas evidência histórica.

Conservar fatos independentes exige independência causal, não proximidade textual. Diante de controle aberto, uma declaração de “lacuna irrelevante para esta consulta” exige justificar que a fronteira não alcança nem modifica o domínio consultado.

## 7. Limites de análise

Limites de tempo, memória, cardinalidade de candidatos, profundidade de contexto ou iterações não são propriedades do programa. Quando uma análise é interrompida ou aproximada por limite, deve publicar `ANALYSIS_LIMIT`, escopo e restante desconhecido. Uma publicação não pode ser apresentada como completa com operações ou candidatos silenciosamente descartados.

Não se exige que toda análise materialize todos os pares possíveis. Representações compactas e consultas sob demanda são permitidas, desde que possuam significado equivalente ou perda de precisão explicitamente declarada.

## 8. Alegações de ausência

“Não há dependência”, “não há escrita”, “não há outro successor” e “este ponto é inalcançável” são alegações positivas que exigem completude no domínio pertinente. Não são defaults de coleções vazias. Falha de integridade deve ser relatada como IR inválida, não como uma dessas ausências.
