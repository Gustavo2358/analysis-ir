# 11 — Rastreabilidade de requisitos pelos dois lados

**Analysis IR 2.0.0 — Normativo**

Cada linha estabelece um requisito de preservação. A ausência de uma informação no produtor ativa o tratamento indicado; não remove a necessidade do consumidor. Os IDs de oráculo referem-se ao [catálogo de conformidade](../conformidade/02-oraculos.md).

| Requisito | Informação exigida do produtor | Representação IR | Observação exigida pelo consumidor | Informação ausente | Oráculos |
| --- | --- | --- | --- | --- | --- |
| B-01 — Identidade | Entidades e ocorrências distinguíveis | IDs próprios e referências fechadas | Joins sem nomes/linhas | Origem opaca permitida; referência quebrada não | O-29, O-30, O-31 |
| B-02 — Estado mutável | Destino e valor de atualização | `assign`, place e expressão | Definições e uso da origem no ponto correto | `havoc`/opaque; não inventar literal | O-01, O-08, O-11 |
| B-03 — Ordem | Avaliação e sequência semânticas | Ordem intrassequência e transferências | Reads antes de writes e antes de chamadas | Controle aberto se ordem não conhecida | O-08, O-23, O-28 |
| B-04 — Seleção | Predicado/abstração e destinos | `branch` e `dispatch` | Successors, reuniões e definições alternativas | Predicado puro unknown ou controle opaco | O-02, O-03, O-05 |
| B-05 — Repetição | Teste, corpo, atualização e entrada | Ciclo explícito | Fixpoint, zero/uma ou mais iterações | Controle aberto, nunca uma execução arbitrária | O-06, O-07, O-17 |
| B-06 — Saída | Retorno, exceção ou término | `return`, `raise`, `halt` | Ausência de fallthrough e escopo correto | Alternativas conservadoras | O-04, O-18, O-19 |
| B-07 — Chamada | Target, argumentos e resultados | `invoke` e modos de passagem | Target antes de efeitos; resultados por outcome | Efeito/contrato aberto | O-20, O-21, O-23, O-24 |
| B-08 — Storage | Associação de objetos e duração | Célula, região, alias ou associação aberta | Strong/weak update e alias | Escopo de storage desconhecido | O-12, O-13, O-14, O-37 |
| B-09 — Fragmentos | Offset, extensão e unidade | Vistas de bytes e cópia por intervalo | Definições por byte/faixa | Intervalo aberto com efeito conservador | O-38, O-39, O-51 |
| B-10 — Conversões | Regra de valor e representação | Conversões/codec explícitos | Nome e conteúdo corretamente interpretados | Unknown de valor, não conversão usual | O-25, O-26, O-40, O-52 |
| B-11 — Input | Disponibilidade e limites da entrada | Coverage e `INPUT_MISSING` | Não confundir vazio com ausência | Inventário partial/unavailable | O-32, O-33, O-45 |
| B-12 — Unknown | Conhecido, não conhecido e escopo | Três envelopes e razões tipadas | Preservação conservadora sem no-op | Escopo máximo quando não delimitável | O-15, O-16, O-34 |
| B-13 — Evidência | Origem e transformação | Origem escrita/derivada/contratual | Explicação de operações e fatos | `UNAVAILABLE`, nunca span fabricado | O-41, O-42, O-43 |
| B-14 — Independência | Limites de influência demonstráveis | Escopos de controle/memória/recurso | Manter fatos realmente independentes | Não alegar independência sem limite | O-15, O-22, O-46 |
| B-15 — Invocação local | Entrada, conclusão e retorno | `control.local@1` | Pareamento e estado compartilhado | Fallback conservador de controle | O-56 a O-60 |
| B-16 — Controle calculado | Tipo e universo de destinos | `control.indirect@1` | CFG inicial sem depender de values | Universo aberto/opaque se não delimitável | O-61 a O-63 |
| B-17 — Recursos | Categoria, ação e nome/operando | ResourceRef e site `invoke` | Dependência por espécie, não só call graph | Restante de recursos desconhecidos | O-35, O-36, O-64 |
| B-18 — Artefatos | Relações sem execução | `ArtifactRelation` | Includes/schema sem CFG fictício | Cobertura estrutural parcial | O-65 |
| B-19 — Extensão | Nova semântica tipada | Operação/capacidade com versão e envelopes | Consumo preciso ou fallback seguro | Incompatibilidade explícita | O-47, O-48 |
| B-20 — Escala | Inventário completo e limites reais | Cardinalidade irrestrita, representação compacta | Consulta sob demanda e resultados honestos | `ANALYSIS_LIMIT` com restante | O-49, O-50, O-68 |
| B-21 — Conhecimento de tipo | Domínio estabelecido ou lacuna localizada, entidades e usos conhecidos | `TypeRef` em objetos, células, locais, expressões, ocorrências e assinaturas | Distinguir tipo/valor desconhecidos de domínio de extensão; preservar usos sem inventar compatibilidade | `unknown_type`, leituras e envelopes; rejeição de precondição concreta não satisfeita | O-69 a O-81 |
| B-22 — Domínio comum não identificado | Identidade/alias exato ou premissa de mesmo domínio com autoridade, sujeitos e `DomainProofScope`; garantia universal quando o sujeito é uma escolha inteira; semântica de cópia sem conversão | `sameDomain` independente de `TypeRef`, escopos estáticos com aplicabilidade/interseção, `assign` e transmissão com captura | Conservar a relação de valor no ponto anterior sem interpretar domínio ausente; verificar escopo sem consultar frontend ou análise de execução | Sem prova aplicável, abstração conserva reads/writes; lacuna compartilhada não prova igualdade | O-77, O-82 a O-85 |

## Critério de aceitação de nova capacidade

O fechamento da edição especializa os requisitos existentes:

| Requisito | Fato do produtor e representação | Observação do consumidor | Ausência/contracaso | Oráculos |
| --- | --- | --- | --- | --- |
| B-01/B-07/B-22 — Contrato fechado | Assinatura por entrada/site, `ContractRef` com autoridade/versão/evidência, efeitos/outcomes/premissas materializados | Interpretar a chamada e seus sujeitos sem serviço externo ou IDs de Java | Restantes e `CONTRACT_UNKNOWN`; posição inexistente é inválida | O-86, O-91 |
| B-08/B-14 — Separação entre bases | Premissa universal `disjoint_storage` com evidência; intervalos da mesma base usam fatos já existentes | Determinar se um efeito limitado pode alcançar outro local | Sem garantia não presumir independência por IDs; associação aberta permanece aberta | O-14, O-15, O-88 |
| B-17/B-18 — Identidade de recurso | Target interno/literal/calculado e relações com identidade própria | Observar nome e site sem resolução fictícia de catálogo | `ResourceId` não substitui target; literal não prova existência | O-24, O-65, O-87 |
| B-12/B-19 — Precondições e fallback | Operação e garantia semântica ou envelope; certificado de Validator é evidência separada | Consumir fatos definidos e registrar limites, sem interpretar tokens de testes | Sem precondição, abstração; extensão não negociada é incompatível | O-89, O-90 |

Uma capacidade proposta DEVE acrescentar ou especializar uma linha dessa matriz e fornecer exemplo positivo, contracaso, comportamento incompleto e impacto em compatibilidade. Se o produtor não consegue fornecer o requisito preciso, deve ser demonstrado que o fallback é conservador. Se o consumidor não precisa da informação, sua inclusão no núcleo deve ser reconsiderada.

A matriz não congela um frontend ou algoritmo. Congela as obrigações observáveis que tornam a fronteira suficiente. Uma alteração de representação é aceitável quando essas obrigações e a política de compatibilidade permanecem satisfeitas.
