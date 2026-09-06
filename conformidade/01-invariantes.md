# Invariantes de conformidade

**Analysis IR 2.0.0 — Normativo**

## 1. Classes de verificação

`Estrutural` designa consistência verificável a partir da publicação. `Semântico` designa obrigação que depende também da interpretação das operações, premissas e, para o produtor, da correspondência com sua entrada. `Derivado` designa obrigação de resultado de um consumidor.

Uma verificação estrutural bem-sucedida não certifica automaticamente os invariantes semânticos. A evidência de conformidade DEVE identificar quais classes foram verificadas e quais dependem de contratos/premissas.

| ID | Classe | Regra obrigatória |
| --- | --- | --- |
| I-01 | Estrutural | Cada identidade é única em seu domínio e namespace completo. |
| I-02 | Estrutural | Toda referência interna aponta para entidade existente e de domínio compatível. |
| I-03 | Estrutural | Cada operação pertence a exatamente uma sequência e unidade. |
| I-04 | Estrutural | Cada sequência termina em exatamente um terminador; não há operação posterior a ele. |
| I-05 | Estrutural | Cada entrada de corpo disponível aponta para label existente na unidade. |
| I-06 | Semântico | Ordem de sequências não implica execução; ordem de operações na sequência é preservada. |
| I-07 | Estrutural | Um label identifica início de sequência, não uma posição intermediária oculta. |
| I-08 | Estrutural | `TypeRef`, provas de domínio e cardinalidades satisfazem as precondições da assinatura. Uma exigência de domínio concreto `T` requer `known(T)`; cópia/transmissão exige `sameDomain` comprovado, sem necessariamente identificar `T`. `unknown_type` sozinho não satisfaz nenhuma dessas provas. Posições/assinaturas parciais seguem o contrato explícito de `invoke`. |
| I-09 | Semântico | Expressão pura não oculta escrita, invocação, transferência ou saída excepcional. |
| I-10 | Semântico | Avaliação e captura antecedem a escrita/efeito correspondente. |
| I-11 | Estrutural | Cada ocorrência de operando é distinguível do objeto e dos demais usos. |
| I-12 | Semântico | Declaração nominal não prova identidade ou independência de armazenamento. |
| I-13 | Estrutural | Associações precisas de vistas têm base, unidade de offset, extensão e codec explícitos. |
| I-14 | Semântico | Strong update elimina apenas localizações comprovadamente e completamente sobrescritas. |
| I-15 | Derivado | Escrita possível ou alternativa conserva definições que podem sobreviver. |
| I-16 | Derivado | Escrita parcial conserva contribuição anterior fora do intervalo modificado. |
| I-17 | Semântico | Inicialização declarada não é repetida em entrada/ativação para a qual não vale. |
| I-18 | Semântico | Return, retorno local, raise e halt mantêm escopos de controle distintos. |
| I-19 | Derivado | Retorno preciso corresponde à invocação que originou o contexto. |
| I-20 | Estrutural | Dispatch tem casos únicos por igualdade semântica e default explícito. |
| I-21 | Semântico | Ramo que termina ou diverge não ganha continuação artificial. |
| I-22 | Semântico | Controle desconhecido não é reduzido a fallthrough ordinário. |
| I-23 | Semântico | Ausência de contrato de efeitos não significa efeito vazio. |
| I-24 | Derivado | Target e argumentos de interação são consultados antes de seus efeitos. |
| I-25 | Semântico | Modo de passagem não substitui contrato de efeitos. |
| I-26 | Estrutural | Toda construção opaca/extensão tem envelopes interpretáveis ou incompatibilidade explícita. |
| I-27 | Semântico | Fallback e operação original não são contados como duas execuções. |
| I-28 | Estrutural | Inventário zero completo é distinto de inventário parcial/indisponível. |
| I-29 | Semântico | Toda ocorrência coberta é publicada ou possui eliminação semanticamente justificada e rastreável. |
| I-30 | Semântico | Unknown value, unknown type, unsupported e input missing não viram ausência silenciosa. Tipo desconhecido conserva entidade, leitura, ocorrência, dependências, proveniência e lacunas aplicáveis, mas não autoriza operação que exige domínio conhecido (I-08). |
| I-31 | Derivado | Resumo de completude não excede seus componentes relevantes no mesmo escopo. |
| I-32 | Estrutural | Cada alegação de precisão possui dimensão e escopo identificáveis. |
| I-33 | Derivado | Conjunto finito com restante aberto não é apresentado como exaustivo. |
| I-34 | Derivado | Valor sobrescrito obrigatoriamente não permanece como candidato corrente sem outra derivação válida. |
| I-35 | Derivado | Ponto inalcançável é distinto de ponto alcançável com valor desconhecido. |
| I-36 | Estrutural | Origem é escrita, derivada, contratual ou explicitamente indisponível; não há span fabricado. |
| I-37 | Semântico | Nomes, mensagens e payload de exibição não governam semântica escondida. |
| I-38 | Estrutural | Análises correlacionam-se com a mesma publicação/revisão e premissas compatíveis. |
| I-39 | Semântico | Limites operacionais não truncam inventário ou candidatos silenciosamente. |
| I-40 | Semântico | Identidade de recurso externo não é inferida de nome nominal sem catálogo/contrato. |
| I-41 | Derivado | Dependência observada é distinta de dependência executável/viável ou execução confirmada. |
| I-42 | Derivado | Extração especializada utiliza os produtos canônicos e não reinterpreta controle por texto. |
| I-43 | Estrutural | Capacidades/extensões requeridas têm identidade e versão explícitas. |
| I-44 | Semântico | Uma extensão mantém significado das operações anteriores ou declara incompatibilidade. |
| I-45 | Semântico | Células de ativação não são confundidas entre contextos recursivos sem abstração declarada. |
| I-46 | Semântico | Leitura/escrita com codec ou limite inválido não recebe conversão/recovery silencioso. |
| I-47 | Semântico | Fronteira de controle aberta influencia todas as consultas potencialmente alcançadas por ela. |
| I-48 | Derivado | Resultados de valores são avaliados no ponto da definição, não por reavaliação tardia de sua origem. |
| I-49 | Estrutural | Cada conhecimento de domínio é `known(T)` ou `unknown_type(u)`; `u` fecha sobre lacuna `TYPE_UNKNOWN`. Objetos/células/aliases exatos compartilham seu `TypeRef`; locais, expressões, ocorrências e assinaturas seguem as regras de obtenção/preservação. |
| I-50 | Semântico | Tipo, valor, storage, binding e falta de suporte a extensão são fatos distintos. `opaque_type` identifica domínio conhecido; desconhecimento não fabrica domínio, compatibilidade, conversão, disjunção ou igualdade. Compartilhar `UncertaintyId` não prova igualdade de domínios/valores. |
| I-51 | Estrutural | `choice` só declara `known(T)` com o mesmo domínio assegurado em todos os candidatos e no restante; caso contrário usa `unknown_type`, conservando os tipos próprios dos candidatos. Escolha vazia fechada não é válida para leitura/escrita. |
| I-52 | Estrutural | Uma prova de `sameDomain` tem derivação finita nas regras normativas e sujeitos/referências fechados. Premissas tipadas têm identidade, autoridade, origem e `DomainProofScope` bem formado conforme 02, §1.4; o site de uso pertence à interseção dos escopos de todas as bases. Vínculos de chamadas e ocorrências de escolhas são explícitos; escopo vazio ou de outro site não sustenta a prova. A operação não prova sua própria precondição. Cadeia contraditória entre domínios concretos distintos é inválida. |
| I-53 | Semântico | Premissas de mesmo domínio são sustentadas pela autoridade declarada em todas as execuções dos sites de seu escopo estático. Uma premissa sobre a ocorrência inteira de `choice` cobre todos os candidatos e todo o restante possível, não só o local selecionado. Não afirmam igualdade de valores, alias, codec ou conversão; não dispensam precondição de domínio concreto nem identificam ativações distintas. |
| I-54 | Derivado | Cópia/transmissão válida por `sameDomain` conserva a relação de captura e as evidências do valor no ponto anterior, mesmo com tipo concreto desconhecido; não vira escrita arbitrária apenas pela lacuna. |
| I-55 | Estrutural | `ContractRef` contém autoridade, versão e origens existentes; desconhecimento usa `CONTRACT_UNKNOWN`. `invoke.signature` referencia exatamente a entrada interna ou materializa a assinatura externa no site. Posições, modos, tipos e restantes fecham sem `ContractId`, lookup ou inventário `contracts`. |
| I-56 | Semântico | O conteúdo de contrato é a assinatura, efeitos, outcomes e premissas materializados; autoridade/versão não fornece fato ausente, não fecha restante e não iguala sujeitos de sites distintos. Limites contrários ao corpo/evidência não são aceitos silenciosamente. |
| I-57 | Estrutural | Targets de interação são internos por `EntryId`, literais ou calculados; `ResourceId` não é target executável. Relação de artefato possui `ArtifactRelationId` fechado e destino estrutural sem avaliação dinâmica. |
| I-58 | Estrutural | `disjoint_storage` tem identidade, autoridade, justificativa e origem; refere pelo menos duas bases existentes distintas, sem campo de escopo seletivo. Outras formas de premissa exigem contrato de extensão, não tokens privados de Validator. |
| I-59 | Semântico | A separação entre bases vale par a par para todas as instâncias simultâneas cobertas pela publicação; não separa aliases da mesma base, resolve escolha aberta, fornece domínio/codec ou afirma ausência de efeitos. |
| I-60 | Estrutural | `InvocationOutcomes` preserva unicidade de normal/tags/catch-all, labels locais e restante; limites por outcome têm chaves únicas. `ControlEnvelope` distingue continuação, salto e saídas; `continue` só pertence ao fallback de operação comum, nunca a terminador. |
| I-61 | Semântico | Certificado/token de precondição não acrescenta semântica ao núcleo. Precondições de pureza, recorte, codec e igualdade de extensão permanecem obrigatórias por suas regras; limite do Validator não redefine validade ou seleciona entradas de `return`. |

## 2. Tratamento de falhas

Falhas estruturais DEVEM produzir diagnóstico `INVALID_IR` com identidade e regra violada. O consumidor não deve repará-las por lookup textual, exclusão de item ou criação de destino fictício.

Uma capacidade desconhecida mas bem envelopada não é `INVALID_IR`: pode ser consumida conservadoramente. Falta de perfil no consumidor é `UNSUPPORTED_CAPABILITY`, não uma falha no programa de origem. Semântica de origem não estabelecida é lacuna do produtor, não motivo para afirmar que a construção não existe.

Diagnósticos semânticos devem preservar o menor escopo comprovado. Se esse escopo for desconhecido, a incerteza é ampliada conservadoramente. A rejeição de uma análise precisa não exige apagar observações independentes ainda sustentadas.

## 3. Conformidade de produtores

Um produtor deve demonstrar invariantes estruturais em todas as publicações e invariantes semânticos por regras de tradução, oráculos e limites explícitos. Declarar premissas não autoriza inventá-las: sua autoridade, origem e impacto devem estar registrados. Uma conclusão condicionada a premissa não pode ser anunciada como incondicional.

## 4. Conformidade de consumidores

Um consumidor deve demonstrar invariantes derivados e os oráculos do perfil alegado. O contrato não prescreve testes internos específicos, mas um resultado que viola um oráculo obrigatório não satisfaz o perfil, ainda que os demais cenários sejam aprovados.
