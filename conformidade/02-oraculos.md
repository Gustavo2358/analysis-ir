# Oráculos de conformidade

**Analysis IR 2.0.0 — Normativo**

## Convenções

Cada oráculo define comportamento de conformidade; não prescreve um framework ou uma implementação. Os exemplos associados são encontrados em [fluxo](../exemplos/01-fluxo-e-valores.md), [memória](../exemplos/02-memoria-e-chamadas.md), [extensões](../exemplos/03-extensoes-e-parcialidade.md) e [conhecimento de tipo](../exemplos/04-conhecimento-de-tipo.md).

Quando se exige um conjunto exato, valem as premissas do cenário, o perfil preciso pertinente e o escopo de entradas explicitado. Consumers apenas conservadores podem produzir resultados mais amplos com limites declarados, mas não satisfazem por isso o perfil de precisão que exige aquele resultado. Os valores textuais exibidos sem aspas nesta tabela continuam literais de texto; IDs RD designam eventos, não valores.

Para invariância de identidade entre representações equivalentes, compara-se por uma correlação explícita de IDs, e não por igualdade de números locais.

### Sub-requisitos por produto

Um ID com sufixo, como `O-77-STRUCT` ou `O-77-SCALAR`, designa somente as assertivas nomeadas por ele. O cenário e os contracasos são os do oráculo-base; uma falha se aplica ao sub-requisito apenas quando contradiz suas assertivas. Referir o ID-base sem sufixo exige todos os sub-requisitos, quando houver. `STRUCT` verifica/preserva fatos publicados, integridade, contratos e controle explícito; não exige calcular valores nem efeitos escalares. Assertivas `INVALID_IR` pertencem ao papel `Validator`; o consumidor estrutural deve recusar alegação válida e conservar diagnóstico, sem reparar a publicação. `SCALAR` exige efeitos/capturas/valores escalares especificados; `REGION` exige a interpretação de memória por intervalos/codecs especificada. Cada implementação declara o produto a que sua evidência corresponde, conforme os perfis.

As projeções estruturais dos oráculos anteriores são delimitadas abaixo. Elas reutilizam os cenários originais; suas assertivas substituem, para a referência com sufixo, qualquer obrigação de RD/PV do oráculo-base.

| ID | Assertivas estruturais obrigatórias |
| --- | --- |
| O-01-STRUCT | Preservar a, b, k e sua ordem, destinos e operandos. |
| O-02-STRUCT | Preservar os dois ramos e suas transferências à continuação comum. |
| O-03-STRUCT | Preservar o caminho falso direto à continuação, sem fabricar escrita nesse caminho. |
| O-04-STRUCT | Preservar `halt` sem successor e os demais destinos explícitos. |
| O-05-STRUCT | Preservar todos os casos/default; rejeitar casos duplicados por igualdade dos literais. |
| O-06-STRUCT | Preservar teste, saída e back-edge, inclusive caminho de zero iterações. |
| O-07-STRUCT | Preservar entrada pelo corpo, teste e back-edge, sem caminho direto da entrada à saída. |
| O-08-STRUCT | Preservar operação de captura, ocorrência `read(@source)` e ponto anterior a ela; não exige calcular A. |
| O-09-STRUCT | Permutar sequências preserva ordem intrassequência, referências e transferências. |
| O-10-STRUCT | Dividir sequência com jump preserva operações, pontos correlacionados e fluxo explícito. |
| O-18-STRUCT | `return` sai da ativação sem fallthrough à sequência seguinte. |
| O-19-STRUCT | `halt` não alcança a continuação normal do invocador como retorno. |
| O-20-STRUCT | Preservar lista de resultados vinculada apenas ao outcome normal; não exige calcular conteúdo de status. |
| O-21-STRUCT | Preservar `any_exception` e restantes de controle não excluídos. |
| O-22-STRUCT | Preservar continuação única e envelope de escrita aberto como fatos independentes; não calcula seus efeitos em valores. |
| O-29-STRUCT | Identidades e referências permanecem separadas por unidade/namespace. |
| O-30-STRUCT | Referência interna inexistente produz `INVALID_IR` com domínio/identidade; não vira recurso externo implícito. |
| O-31-STRUCT | Recusar composição de revisões sem correlação/compatibilidade explícita. |
| O-32-STRUCT | Distinguir inventário zero completo de inventário indisponível. |
| O-33-STRUCT | Preservar a construção e seus envelopes entre as operações conhecidas. |
| O-34-STRUCT | Preservar fronteira aberta que admite todo `ControlScope`, sem fechar o fluxo no successor aparente. |
| O-41-STRUCT | Preservar origens e regras de derivação, sem spans fabricados. |
| O-42-STRUCT | Operações distintas mantêm IDs distintos apesar da origem comum. |
| O-43-STRUCT | Distinguir origem da declaração e origem da ocorrência de uso. |
| O-44-STRUCT | Alterar texto de exibição não muda identidades, referências, operações, contratos ou controle. |
| O-45-STRUCT | Preservar site literal e inventário parcial; não afirmar exclusividade global de dependências. |
| O-46-STRUCT | Preservar site no inventário e distinguir presença de alcançabilidade no controle fechado do cenário. |
| O-47-STRUCT | Conservar operação/envelope e modo de consumo; escolher uma representação executável sem duplicar original/fallback. |
| O-48-STRUCT | Registrar incompatibilidade quando capacidade requerida não possui interpretação nem envelope compatível. |

## O-01 — Sobrescrita linear

**Cenário:** X-01.

**Resultado obrigatório:** No ponto anterior à chamada, RD contém apenas b e PV contém apenas B, com restante fechado.

**Falha a detectar:** Manter a ou A como definição/valor corrente preciso.

## O-02 — Diamond com atualização nos dois ramos

**Cenário:** X-02.

**Resultado obrigatório:** RD={b,c}; PV={B,C}; o CFG reconverge na continuação e mantém as duas alternativas.

**Falha a detectar:** Conservar A; escolher só um ramo; exigir conhecer o predicado para formar os dois successors.

## O-03 — Ramo falso sem escrita

**Cenário:** X-03.

**Resultado obrigatório:** RD={a,b}; PV={A,B}. O destino falso é a continuação comum.

**Falha a detectar:** Eliminar a como se todos os caminhos escrevessem.

## O-04 — Ramo terminante

**Cenário:** X-04.

**Resultado obrigatório:** Somente c alcança k. Não existe aresta de halt para a continuação.

**Falha a detectar:** Unir B ao target da chamada ou criar reconvergência artificial.

## O-05 — Dispatch e default

**Cenário:** X-05.

**Resultado obrigatório:** Casos ONE/TWO e default OTHER permanecem; casos duplicados por valor são rejeitados.

**Falha a detectar:** Omitir default, usar ordem da tabela para resolver duplicatas ou assumir seletor dentro da faixa.

## O-06 — Pré-teste

**Cenário:** X-06.

**Resultado obrigatório:** RD={a,b}; PV={A,B}; existe caminho de zero iterações.

**Falha a detectar:** Exigir execução do corpo ou omitir a contribuição de b através do back-edge.

## O-07 — Pós-teste

**Cenário:** X-07.

**Resultado obrigatório:** RD={b}; PV={B}; nenhum caminho até k evita o corpo.

**Falha a detectar:** Adicionar definição de entrada como se o laço pudesse executar zero vezes.

## O-08 — Captura de valor

**Cenário:** X-08.

**Resultado obrigatório:** O valor de target é A, lido em before(capture), apesar de source valer B depois.

**Falha a detectar:** Reavaliar a expressão da definição no ponto da chamada.

## O-09 — Permutação de sequências

**Cenário:** Metamórfico de X-02.

**Resultado obrigatório:** Permutar a ordem física das sequências, mantendo labels e transferências, conserva resultados e comportamento.

**Falha a detectar:** Criar execução entre sequências pela ordem física da publicação.

## O-10 — Divisão de sequência

**Cenário:** Metamórfico de X-01.

**Resultado obrigatório:** Dividir após a com jump explícito preserva valores e definições nos pontos originais, sob o mapeamento de identidade.

**Falha a detectar:** Mudar efeitos ou perder a correlação dos pontos por exigir blocos básicos fixos na IR.

## O-11 — Leitura anterior à escrita

**Cenário:** Atribuição autorreferente.

**Resultado obrigatório:** Partindo de x=A, assign(x,concat(read(x),B)) produz AB; o read usa a definição anterior.

**Falha a detectar:** Ler o resultado parcial da própria operação ou construir ciclo de valor sem justificativa.

## O-12 — Alias exato

**Cenário:** X-11.

**Resultado obrigatório:** Escrita em y elimina a definição anterior de x porque ambos compartilham a célula.

**Falha a detectar:** Tratar ObjectId distintos como armazenamento independente.

## O-13 — Destino alternativo

**Cenário:** X-12.

**Resultado obrigatório:** x admite A/B e y admite D/B; a escrita é em uma alternativa e exige atualização fraca em cada célula.

**Falha a detectar:** Escolher a primeira alternativa ou sobrescrever ambas obrigatoriamente.

## O-14 — Associação de armazenamento aberta

**Cenário:** Objeto com unknown(scope).

**Resultado obrigatório:** A análise conserva alias/efeito aberto no escopo e não afirma disjunção apenas por IDs.

**Falha a detectar:** Converter o objeto em célula privada nova para obter resultado exato.

## O-15 — Efeito limitado independente

**Cenário:** X-14.

**Resultado obrigatório:** Target conserva KNOWN e restante fechado porque a escrita está limitada a scratch comprovadamente disjunto.

**Falha a detectar:** Contaminar sem necessidade ou, no contracaso sem prova de disjunção, conservar a mesma precisão.

## O-16 — Must versus may

**Cenário:** X-13.

**Resultado obrigatório:** Havoc obrigatório remove A como candidato corrente; havoc possível conserva A e acrescenta restante.

**Falha a detectar:** Usar a mesma transferência para os dois casos.

## O-17 — Retorno por ciclo de definições

**Cenário:** X-06 com ordem de análise adversarial.

**Resultado obrigatório:** A solução estável contém a definição do corpo no ponto de saída, independentemente da ordem física dos labels.

**Falha a detectar:** Aceitar resultado de uma passagem textual que não propagou o back-edge.

## O-18 — Return não tem fallthrough

**Cenário:** Unidade com return antes de outra sequência.

**Resultado obrigatório:** Return sai da ativação; a sequência seguinte só é alcançável por alguma transferência explícita independente.

**Falha a detectar:** Prosseguir por ordem textual após return.

## O-19 — Halt não é return

**Cenário:** Chamador com continuação e callee que termina.

**Resultado obrigatório:** Halt no comportamento chamado encerra a execução pertinente e não alcança a continuação normal.

**Falha a detectar:** Tratar término da execução como conclusão normal de subrotina.

## O-20 — Resultado apenas no retorno normal

**Cenário:** X-18 com status inicialmente 0 e efeitos none.

**Resultado obrigatório:** Após retorno normal, status tem definição de resultado; após exceção, permanece a definição anterior 0, salvo outro efeito explicitado.

**Falha a detectar:** Aplicar resultado normal a todos os outcomes.

## O-21 — Exceções não conhecidas

**Cenário:** Invoke com contrato de controle parcial.

**Resultado obrigatório:** Any_exception ou restante aberto conserva resultados não excluídos pelo contrato.

**Falha a detectar:** Inferir ausência de exceções de uma lista não exaustiva vazia.

## O-22 — Controle fechado e efeitos abertos

**Cenário:** Opaque com único normal e may_write visível.

**Resultado obrigatório:** O successor normal é conhecido, enquanto valores afetados permanecem abertos. As duas dimensões não são colapsadas.

**Falha a detectar:** Promover efeitos a exatos porque controle é fechado ou bloquear estrutura já conhecida sem necessidade.

## O-23 — Target antes dos efeitos da chamada

**Cenário:** X-18 adaptado com target x=A e chamada que pode escrever x.

**Resultado obrigatório:** Em before(invoke), o target é A. Em after(normal), a escrita possível pode abrir os valores seguintes.

**Falha a detectar:** Aplicar efeitos da chamada retroativamente ao seu próprio target.

## O-24 — Alvo literal sem catálogo

**Cenário:** X-17.

**Resultado obrigatório:** O nome literal e namespace são observáveis sem dataflow; identidade de artefato externo permanece não confirmada.

**Falha a detectar:** Inventar artifact ID, mudar caixa ou exigir catálogo apenas para conservar o nome.

## O-25 — Texto com comprimento normalizado

**Cenário:** X-20 e variante fit_text(A,4,space).

**Resultado obrigatório:** Truncamento produz ABCD; preenchimento produz A seguido de três espaços. Não há trim implícito.

**Falha a detectar:** Preservar texto original longo ou remover padding sem contrato.

## O-26 — Decimal exato e arredondamento

**Cenário:** X-20 e empate 1,35.

**Resultado obrigatório:** 1,25 para uma casa half-even produz 1,2; 1,35 produz 1,4; toward_zero deve seguir sua regra distinta.

**Falha a detectar:** Delegar a resultado ambiental de arredondamento ou confundir representação física e valor lógico.

## O-27 — Predicado desconhecido com reads conhecidos

**Cenário:** Branch de unknown(bool,dependencies=[x],remainingReads=none).

**Resultado obrigatório:** Os dois destinos são conservados e a leitura de x é observável; não há escrita nem chamada implícita.

**Falha a detectar:** Perder o read, escolher um ramo ou ocultar efeito impuro em expressão.

## O-28 — Captura compartilhada para atualização simultânea

**Cenário:** Troca de x=A e y=B com auxiliar capturado.

**Resultado obrigatório:** Normalização equivalente produz x=B e y=A. A sequência ingênua x=y; y=x não é equivalente.

**Falha a detectar:** Duplicar/reordenar leitura através de uma atualização que muda seu valor.

## O-29 — Namespaces por unidade

**Cenário:** X-27.

**Resultado obrigatório:** Definições e targets FIRST/SECOND permanecem independentes apesar de IDs locais iguais.

**Falha a detectar:** Unir fatos pelo fragmento local de identidade.

## O-30 — Fechamento de referências

**Cenário:** Referência interna a label/objeto inexistente.

**Resultado obrigatório:** A publicação é INVALID_IR com referência e domínio identificados. Recurso externo tipado é outra forma válida.

**Falha a detectar:** Recuperar destino pelo nome, criar objeto fictício ou descartar a referência.

## O-31 — Mistura de publicações

**Cenário:** Resultado RD de uma revisão usado com outra.

**Resultado obrigatório:** A composição deve recusar ou exigir demonstração explícita de compatibilidade/migração.

**Falha a detectar:** Associar fatos de revisões distintas porque números locais coincidem.

## O-32 — Zero versus indisponibilidade

**Cenário:** Duas publicações sem operações.

**Resultado obrigatório:** Inventory complete pode afirmar zero; inventory unavailable não pode afirmar ausência de programa/efeitos.

**Falha a detectar:** Tratar as duas listas vazias como sucesso equivalente.

## O-33 — Construção fora da capacidade entre operações conhecidas

**Cenário:** X-25 ou operação unsupported entre assign e invoke.

**Resultado obrigatório:** A construção permanece no inventário e seus envelopes afetam as consultas pertinentes.

**Falha a detectar:** Publicar apenas assign e invoke como se fossem adjacentes sem interferência.

## O-34 — Fronteira aberta de controle

**Cenário:** X-25.

**Resultado obrigatório:** A fronteira admite todo o ControlScope declarado e impede completude de consultas afetadas.

**Falha a detectar:** Ligar apenas ao fim ou ao successor aparente e fechar o CFG.

## O-35 — Categorias de recurso

**Cenário:** X-21.

**Resultado obrigatório:** Fatos distinguem read/table e call/service, com sites, targets e namespaces próprios.

**Falha a detectar:** Converter todos os usos em chamada a programa.

## O-36 — Literal em um ramo e input no outro

**Cenário:** Variante de X-02: b literal B; c havoc.must(target).

**Resultado obrigatório:** PV conserva B como candidato enumerado e marca unknownRemainder=true.

**Falha a detectar:** Apagar o candidato B por colapsar tudo a desconhecido, ou publicar conjunto fechado {B}.

## O-37 — Fatia conhecida em região parcialmente desconhecida

**Cenário:** X-15.

**Resultado obrigatório:** Destino [4,12) é conhecido; byte 3 e pacote completo não são.

**Falha a detectar:** Contaminar a fatia sem alias/sobreposição ou declarar o pacote inteiro conhecido.

## O-38 — Escritas sobre vistas sobrepostas

**Cenário:** Região com vistas [0,4) e [2,6).

**Resultado obrigatório:** Uma escrita em [2,6) modifica a sobreposição [2,4), conserva [0,2) e não elimina contribuições fora do intervalo.

**Falha a detectar:** Aplicar kill por nome de vista ou por toda a região independentemente de intervalo.

## O-39 — Offset calculado

**Cenário:** Acesso de 4 bytes em região de 8 com offset em {0,4}.

**Resultado obrigatório:** O endereço lê o seletor e representa os dois intervalos. Escritas são fracas em cada intervalo quando a escolha não é conhecida.

**Falha a detectar:** Escolher offset 0, ignorar o read do índice ou alegar write obrigatório nos dois intervalos.

## O-40 — Codec desconhecido

**Cenário:** Vista com codec não interpretado, bytes conhecidos.

**Resultado obrigatório:** A cópia bruta pode permanecer precisa; a decodificação para valor/nome permanece aberta ou indisponível.

**Falha a detectar:** Assumir ASCII ou outra codificação habitual.

## O-41 — Origem derivada

**Cenário:** Normalização de uma operação em vários jumps/assigns.

**Resultado obrigatório:** Todos os derivados referenciam origem e regra de derivação, sem fingir spans escritos individuais.

**Falha a detectar:** Fabricar linhas-fonte para instruções auxiliares.

## O-42 — Clones e origem compartilhada

**Cenário:** Duas operações distintas derivadas da mesma origem.

**Resultado obrigatório:** IDs de operação são distintos; origem comum é permitida; evidências de sites podem ser agregadas sem apagar ocorrências.

**Falha a detectar:** Usar a origem como ID único e colapsar execuções/definições diferentes.

## O-43 — Operando e declaração com origens distintas

**Cenário:** Read de objeto declarado em outro artefato.

**Resultado obrigatório:** O consumidor explica separadamente a origem da declaração e a origem da ocorrência de uso.

**Falha a detectar:** Substituir ambas por uma única localização de conveniência.

## O-44 — Texto de diagnóstico e exibição

**Cenário:** Alterar mensagens e nomes de exibição sem mudar fatos.

**Resultado obrigatório:** Resultados semânticos permanecem iguais; origens/metadados de exibição podem mudar.

**Falha a detectar:** Mudar branch, binding ou efeito por prefixo de mensagem ou nome.

## O-45 — Inventário incompleto com literal observado

**Cenário:** X-28.

**Resultado obrigatório:** O site KNOWN permanece OBSERVED; não se afirma que seja a única dependência global.

**Falha a detectar:** Apagar o literal ou declarar inventário completo.

## O-46 — Observado versus alcançável

**Cenário:** Acrescentar invoke literal em sequência sem predecessor nem entrada.

**Resultado obrigatório:** O site permanece no inventário OBSERVED; um CFG fechado pode provar ausência de MAY_EXECUTE para as entradas escolhidas.

**Falha a detectar:** Confundir presença sintática com execução ou apagar observações do produto de inventário.

## O-47 — Extensão desconhecida com envelope

**Cenário:** X-26.

**Resultado obrigatório:** O consumidor aplica fallback, abre valor do resultado e mantém controle/efeitos delimitados; declara consumo conservador.

**Falha a detectar:** Ignorar operação, conservar valor antigo preciso ou executar original mais fallback.

## O-48 — Extensão sem contrato compatível

**Cenário:** Capacidade requerida sem interpretação nem envelope.

**Resultado obrigatório:** O escopo afetado recebe incompatibilidade explícita; não há alegação de conformidade precisa.

**Falha a detectar:** Aceitar silenciosamente ou tratar campos desconhecidos como vazios.

## O-49 — Cardinalidade de ocorrências

**Cenário:** N atribuições e M chamadas independentes, inclusive N≠M e zero.

**Resultado obrigatório:** O inventário preserva exatamente N e M quando cobertos, sem pareamento ou limite de um.

**Falha a detectar:** Selecionar primeiro/último ou exigir uma atribuição para cada chamada.

## O-50 — Limite de análise

**Cenário:** X-29 sob limite finito.

**Resultado obrigatório:** O resultado declara aproximação/ANALYSIS_LIMIT e restante; o inventário IR não é truncado.

**Falha a detectar:** Anunciar um prefixo finito de valores como conjunto exaustivo.

## O-51 — Cópia com sobreposição

**Cenário:** X-16.

**Resultado obrigatório:** O conteúdo final é ABABCD; contribuições por intervalo correspondem à captura anterior.

**Falha a detectar:** Copiar progressivamente valores já alterados e obter outro conteúdo.

## O-52 — Violação de domínio de codec

**Cenário:** Escrita de caractere fora de ASCII em vista ASCII exata.

**Resultado obrigatório:** A operação não recebe alegação precisa total; deve haver tratamento explícito de erro/abstração, ou rejeição se a publicação contradiz sua precondição.

**Falha a detectar:** Substituir por interrogação, truncar ou aceitar silenciosamente.

## O-53 — Intervalo vazio

**Cenário:** copy_bytes de comprimento zero com endereços puros válidos.

**Resultado obrigatório:** Não há definição nova de conteúdo; cálculos de endereço e seus reads continuam observáveis.

**Falha a detectar:** Apagar reads de índices ou inventar escrita de um byte.

## O-54 — Escrita parcial possível

**Cenário:** May_write sobre subintervalo de uma região.

**Resultado obrigatório:** Definições antigas podem sobreviver; a incerteza limita-se ao intervalo e suas vistas sobrepostas.

**Falha a detectar:** Executar kill obrigatório ou contaminar intervalo comprovadamente disjunto.

## O-55 — Reconstrução por fragmentos

**Cenário:** Região ABCDEF com escrita obrigatória XY em [2,4).

**Resultado obrigatório:** Leitura completa produz ABXYEF; RD identifica a origem antiga em [0,2)/[4,6) e nova em [2,4).

**Falha a detectar:** Escolher só uma definição para o objeto inteiro ou preservar C/D como bytes correntes.

## O-56 — Retornos de chamadas locais

**Cenário:** X-23.

**Resultado obrigatório:** C1 retorna a after_first e C2 a after_second; memória é compartilhada e x não é reinicializado.

**Falha a detectar:** Misturar continuações ou criar nova ativação de células em local.invoke.

## O-57 — Portas de conclusão distintas

**Cenário:** X-24.

**Resultado obrigatório:** Short retorna em middle com B; long passa pelo default de middle e retorna em end_range com C.

**Falha a detectar:** Tratar toda boundary como retorno incondicional.

## O-58 — Somente frame do topo

**Cenário:** Frame externo aguarda porta A; interno aguarda B; executar boundary A.

**Resultado obrigatório:** A regra segue default e preserva ambos os frames porque o topo não corresponde; não procura o externo.

**Falha a detectar:** Remover frame externo através do interno sem operação de unwind.

## O-59 — Resume sem frame

**Cenário:** local.resume em entrada com pilha vazia.

**Resultado obrigatório:** Há saída excepcional invalid_local_return, não retorno normal nem fallthrough.

**Falha a detectar:** Ignorar o erro ou usar continuação arbitrária.

## O-60 — Unwind explícito

**Cenário:** Dois frames e local.unwind(1,destination).

**Resultado obrigatório:** Só o frame do topo é removido; jump ordinário não remove nenhum. Contagem maior que a profundidade produz invalid_local_unwind.

**Falha a detectar:** Abandonar contextos por inferência textual ou ignorar contagem inválida.

## O-61 — CFG inicial de transferência indireta

**Cenário:** X-22 antes de valores refinados.

**Resultado obrigatório:** O CFG admite todos os labels do universo S, sem depender de RD/PV prévio.

**Falha a detectar:** Usar apenas o primeiro literal atribuído ou criar dependência circular que impede CFG conservador.

## O-62 — Refinamento de destinos

**Cenário:** X-22 com PV(route)={q}.

**Resultado obrigatório:** A revisão refinada pode restringir j a q; os resultados derivados identificam a revisão utilizada.

**Falha a detectar:** Misturar RD antigo e CFG novo sem invalidar/revalidar resultados dependentes.

## O-63 — Destino fora do tipo

**Cenário:** Literal de label não pertencente ao universo declarado.

**Resultado obrigatório:** A publicação é inválida ou deve ser remodelada com universo/envelope suficiente antes de publicar.

**Falha a detectar:** Descartar o destino exterior mantendo alegação de controle fechado.

## O-64 — Protocolo sobre argumento

**Cenário:** X-15.

**Resultado obrigatório:** Com contrato especializado aplicável e declarado na consulta derivada, a fatia e a regra de padding permitem target PGMA; sem contrato, só ROUTER e os valores do argumento são sustentados.

**Falha a detectar:** Inferir dependência indireta por offset habitual ou interpretar protocolo dentro do core.

## O-65 — Dependência estrutural separada

**Cenário:** X-30.

**Resultado obrigatório:** Includes e uses_schema não geram arestas de CFG; table read conserva seu próprio site executável.

**Falha a detectar:** Transformar inclusão em chamada ou inferir acessos de tabela apenas do schema.

## O-66 — Dois produtores independentes

**Cenário:** Duas publicações equivalentes por renomeação de IDs próprios.

**Resultado obrigatório:** Consumers obtêm observações equivalentes sob a correlação de identidades, sem precisar do mesmo produto de entrada.

**Falha a detectar:** Exigir classes, árvore, nomes de campos ou IDs do produtor original.

## O-67 — Evolução de precisão

**Cenário:** Substituir opaque por operação precisa em nova revisão.

**Resultado obrigatório:** O significado da operação precisa é estável; candidatos espúrios podem desaparecer, e efeitos sobre compatibilidade são declarados.

**Falha a detectar:** Exigir crescimento monotônico de todos os resultados ou alterar operação antiga sem versão.

## O-68 — Consulta sob demanda

**Cenário:** Comparar resultado por site com análise integral sob iguais premissas/perfis.

**Resultado obrigatório:** Os resultados são equivalentes no escopo ou explicam diferença de precisão/limite. Fato literal local não exige catálogo/CFG global irrelevante.

**Falha a detectar:** Publicar resultado mais forte porque uma região relevante deixou de ser visitada sob demanda.

## O-69 — Domínio conhecido do núcleo

**Cenário:** X-31, objeto `@name` com `known(text)` e atribuição de literal textual.

**Resultado obrigatório:**

**O-69-STRUCT:** Objeto, célula, `object(@name)`, `read` e ocorrência conservam `known(text)`; a atribuição satisfaz I-08.

**O-69-SCALAR:** Após a escrita, o literal A é sustentado como valor corrente de `@name`.

**Falha a detectar:** Perder o domínio ao propagá-lo entre declaração, local e operando, ou exigir frontend para identificá-lo.

## O-70 — Domínio de extensão não é tipo desconhecido

**Cenário:** X-31, objeto `@token` com `known(opaque_type(example.token,1))`, inclusive consumidor sem interpretação interna do domínio. Variante: cópia entre duas células desse mesmo domínio sob contrato de consumo conservador; contracaso: `concat(read(@token),text("A"))`.

**Resultado obrigatório:**

**O-70-STRUCT:** O domínio e a versão permanecem identificados em locais/operandos; a cópia tem prova de domínio comum e a concatenação é `INVALID_IR`. Preservar contrato/capacidade e modo de consumo; falta de suporte não vira `TYPE_UNKNOWN`.

**O-70-SCALAR:** Interpretar leitura, escrita de `havoc` e captura da cópia de valor sem interpretar a estrutura interna de tokens; `havoc` abre conteúdo, a cópia conserva o valor capturado e nenhum deles troca o domínio.

**Falha a detectar:** Tratar domínio opaco como tipo ausente, aceitar texto implicitamente, inventar igualdade não definida ou usar `opaque_type("unknown",1)` para uma lacuna de domínio.

## O-71 — Entidade com domínio não estabelecido

**Cenário:** X-31, objeto `@untyped` com `unknown_type(u)` e célula identificada. Variantes negativas: remover a lacuna `u` ou alterar seu código para um que não seja `TYPE_UNKNOWN`.

**Resultado obrigatório:**

**O-71-STRUCT:** O objeto e a célula permanecem presentes com origem e associação conhecidas; tipo desconhecido não implica storage/binding desconhecidos. As variantes negativas são `INVALID_IR` por I-02/I-49. Este oráculo não acrescenta assertiva escalar.

**Falha a detectar:** Apagar declaração, fabricar célula por causa do tipo, perder a lacuna ou aceitar referência de tipo sem significado.

## O-72 — Leitura de tipo desconhecido em operação neutra

**Cenário:** X-32. Variante metamórfica: duas ocorrências de `read(@x)` em `knownOperands`, cada uma referenciada no envelope.

**Resultado obrigatório:**

**O-72-STRUCT:** Conservar `ObjectId`, `StorageId`, `TypeRef`, lacunas/razões e origens da declaração e de cada ocorrência, além do envelope sem escrita e da continuação declarada. Referir uma ocorrência no envelope não duplica seu ID; duas ocorrências distintas não são fundidas.

**O-72-SCALAR:** Os efeitos contêm as leituras declaradas, cada ocorrência contada uma vez, e nenhuma escrita; a falta de tipo não apaga a leitura nem altera o estado.

**Falha a detectar:** Substituir por `nop`, apagar a leitura, inventar escrita/recurso ou perder a distinção entre uso e declaração.

## O-73 — Tipo desconhecido em aritmética

**Cenário:** X-34, `add(read(@x),int(1))`; variantes `neg(read(@x))` e ambos argumentos de tipo desconhecido com a mesma lacuna.

**Resultado obrigatório:**

**O-73-STRUCT:** Os contracasos são `INVALID_IR` por I-08, inclusive com prova de `sameDomain` entre argumentos desconhecidos. Não inferir domínio do operador, outro argumento ou lacuna. `add(unknown(known(int)),int(1))` é válido com resultado `known(int)`.

**O-73-SCALAR:** No caso válido, o resultado numérico permanece não determinado, com restante de valor aberto, sem fabricar literal.

**Falha a detectar:** Interpretar “tipo desconhecido” como dispensa de validação, assumir inteiro ou considerar as variantes positivas como tipo desconhecido.

## O-74 — Tipo desconhecido em texto e booleanos

**Cenário:** X-34, `concat(read(@x),text("A"))`, `not(read(@x))` e variante `branch read(@x)`.

**Resultado obrigatório:**

**O-74-STRUCT:** Cada contracaso é `INVALID_IR` por I-08 mesmo com `sameDomain` entre argumentos. A variante booleana de X-33 é válida por `known(bool)` e pureza estabelecidos; conserva a ocorrência da dependência de tipo desconhecido e ambos os destinos.

**O-74-SCALAR:** A variante válida lê a dependência, não escreve memória e admite os dois valores booleanos, sem converter o domínio da dependência.

**Falha a detectar:** Assumir `text`/`bool`, converter silenciosamente a dependência ou confundir desconhecimento do predicado com desconhecimento de seu tipo.

## O-75 — Valor desconhecido de domínio conhecido

**Cenário:** X-33, `p = unknown(known(text),[],none,v)`; variante de atribuição a célula `known(text)` em X-34.

**Resultado obrigatório:**

**O-75-STRUCT:** Conservar `known(text)`, expressão `unknown` e razão de valor; validar a atribuição sem criar lacuna `TYPE_UNKNOWN`.

**O-75-SCALAR:** Valor não determinado com restante textual aberto; avaliações distintas não têm igualdade garantida. A atribuição captura sua avaliação e sobrescreve o destino conforme a memória.

**Falha a detectar:** Transformar `unknown` em literal, apagar a expressão, abrir o domínio sem motivo ou rejeitar uma operação apenas pelo valor desconhecido.

## O-76 — Valor e domínio desconhecidos

**Cenário:** X-33, `q = unknown(unknown_type(u),[read(@x)],none,v)`.

**Resultado obrigatório:**

**O-76-STRUCT:** Distinguir lacunas de tipo/valor e preservar dependência, ocorrência, origem e `remainingReads=none`. Não derivar `sameDomain` do ID compartilhado.

**O-76-SCALAR:** Preservar a leitura conhecida sem outras leituras; o valor/domínio não ficam fechados por `remainingReads=none`. Não igualar valores de resultado/dependência nem apresentar conjunto vazio fechado em ponto alcançável.

**Falha a detectar:** Substituir a referência de tipo pela razão de valor, perder dependências, inventar um domínio padrão ou interpretar ausência de candidatos como conjunto vazio fechado em ponto alcançável.

## O-77 — Assign exige compatibilidade; abstração preserva a leitura

**Cenário:** X-34, tentativas sem prova de mesmo domínio entre células distintas; `assign(@x,read(@x))`; alias exato e cópia com premissa de X-37; abstração `transfer` de X-34.

**Resultado obrigatório:**

**O-77-STRUCT:** Autoatribuição, alias exato e cópia com premissa aplicável satisfazem I-08 por `sameDomain` sem tipo concreto. Cópia entre sujeitos distintos sem prova, mesmo compartilhando lacuna, é `INVALID_IR`, assim como domínios concretos contraditórios. A abstração conserva operandos, envelope e origens; nenhum `assign` válido é substituído apenas por falta de tipo concreto.

**O-77-SCALAR:** `assign` válido conserva leitura anterior, evento de definição e valor capturado, inclusive na autoatribuição. Na cópia de X-37, alterar `@x` depois de `capture` não altera o valor capturado em `@y`. `transfer` de X-34, sem semântica de cópia estabelecida, conserva leitura e sobrescrita com conteúdo aberto, sem inventar identidade de valores.

**Falha a detectar:** Rejeitar cópia comprovada por falta de tipo concreto, criar compatibilidade por igualdade de lacunas, substituir cópia por escrita arbitrária ou apagar a leitura da abstração.

## O-78 — Choice e binding preservam tipos próprios

**Cenário:** X-35. Variantes: escolha fechada só de locais `known(text)`; escolha vazia fechada; candidato com tipo desconhecido; binding físico `unknown(scope,reason)` em objeto de tipo `known(text)`.

**Resultado obrigatório:**

**O-78-STRUCT:** Escolha homogênea fechada conserva `known(text)`; heterogêneas ou sem garantia para o restante usam `unknown_type` preservando candidatos/domínios. `known(text)` sem garantia viola I-51; escolha vazia fechada é inválida para leitura/escrita. Binding físico desconhecido conserva `known(text)` do objeto.

**O-78-SCALAR:** A leitura admite todos os candidatos e o restante quando presente; a falta de tipo não seleciona candidato, apaga leitura nem fecha binding físico aberto.

**Falha a detectar:** Escolher o primeiro candidato, tratar desconhecimento como conversão/união de tipos, perder tipos conhecidos ou misturar lacuna nominal/física com lacuna de domínio.

## O-79 — Assinaturas parciais e precondições concretas

**Cenário:** X-36 sem prova de transmissão precisa e seus contracasos; variantes positivas com domínio conhecido comum ou provas de X-38.

**Resultado obrigatório:**

**O-79-STRUCT:** Preservar posições, modos, `TypeRef`, ocorrências, target e outcomes. Sem `sameDomain`, X-36 não alega transmissão precisa; com prova aplicável, X-38 é válido. Transmissão precisa sem prova para parâmetro/resultado, inclusive para `known(int)`/`known(text)`, viola I-08. Tipo de posição desconhecido não apaga aridade nem torna assinatura vazia.

**O-79-SCALAR:** Resultado normal é escrito somente nesse outcome. A chamada conservadora de X-36 produz conteúdo aberto; a transmissão de X-38 conserva o valor capturado e devolvido pelo corpo. Prova apenas de mesmo domínio não afirma que todo resultado copie o argumento.

**Falha a detectar:** Aceitar precondição concreta sem prova, tratar posição desconhecida como polimorfismo, fabricar argumento/resultado ou aplicar resultado normal na exceção.

## O-80 — Havoc não muda o conhecimento do domínio

**Cenário:** Célula exata `@x : unknown_type(u)` com conteúdo de entrada desconhecido; executar `havoc.must @x`. Variante independente: `havoc.may {@x}`. Demais locais são comprovadamente disjuntos.

**Resultado obrigatório:**

**O-80-STRUCT:** Preservar espécie `havoc.must`/`havoc.may`, destino/escopo, `unknown_type(u)`, razão de escrita, identidade e fatos de independência.

**O-80-SCALAR:** `must` sobrescreve obrigatoriamente a célula; `may` admite ausência de escrita. Nenhum muda domínio/extensão nem contamina local comprovadamente disjunto apenas por falta de tipo.

**Falha a detectar:** Escolher domínio novo, converter o conteúdo, apagar escrita comprovada ou abrir todos os efeitos apenas por desconhecer o tipo.

## O-81 — Domínio lógico e codec são conhecimentos distintos

**Cenário:** Três variantes com mesmo intervalo físico válido e conhecido: vista com `text.ascii@1`; vista de domínio `known(text)` mas codec não interpretado; vista cujo domínio lógico e interpretação não são estabelecidos, com `unknown_type(u)`. Nas variantes incompletas, o cenário garante leitura total sem efeitos excepcionais.

**Resultado obrigatório:**

**O-81-STRUCT:** Conservar respectivamente `known(text)`; `known(text)` com `CODEC_UNKNOWN`; `unknown_type(u)` com lacuna de interpretação. Preservar intervalo e ocorrência de leitura. Rejeitar alegação de domínio desconhecido contraditória com o codec declarado que estabelece `text`.

**O-81-REGION:** Preservar bytes/fatos físicos e cópia bruta sob as precondições de `copy_bytes`. Sem totalidade, a leitura exige envelope de erro/controle. `sameDomain` não autoriza escrita precisa sem codec/domínio de codificação aplicáveis.

**Falha a detectar:** Fazer codec desconhecido implicar sempre tipo desconhecido, escolher ASCII implicitamente, alegar `unknown_type` apesar de codec que estabelece domínio conhecido ou apagar fatos físicos independentes.

## O-82 — Provas de mesmo domínio são fatos com escopo

**Cenário:** X-37. Variantes: premissa ausente; `scope=operation(change)` em vez de `operation(capture)`; sujeito inexistente; cadeia circular sem base; cadeia ligando `known(int)` e `known(text)` por sujeito desconhecido; bases independentes combinadas por simetria/transitividade no escopo comum. O-85 detalha a álgebra de escopos.

**Resultado obrigatório:**

**O-82-STRUCT:** Admitir as derivações finitas válidas por identidade, alias exato, leitura e premissa/combinação aplicável. Rejeitar referências quebradas, contradições e uso de prova ausente, circular ou fora do escopo. Preservar sujeitos, autoridade, origem e lacunas sem unificar `UncertaintyId`. As variantes inválidas violam I-08/I-52; `sameDomain` não satisfaz precondição concreta de `add`, `concat`, `not` ou `eq`.

**O-82-SCALAR:** Cópias válidas conservam relação de captura; igualdade de domínio sem operação de cópia não iguala conteúdos de células distintas. Autoatribuição continua uma leitura e definição, sem virar desaparecimento de ocorrência.

**Falha a detectar:** Usar a própria operação como prova, transformar lacuna em variável de unificação, promover tipo por conveniência ou perder cópia já comprovada.

## O-83 — Provas em parâmetros, retornos e resultados

**Cenário:** X-38 e suas variantes de `value`, `copy`, `reference`, segunda chamada e remoção de vínculos.

**Resultado obrigatório:**

**O-83-STRUCT:** Validar cada vínculo de domínio no site de uso definido em 02, §1.4; preservar modos e origens. `p_in`/`p_out` aplicam-se a `invocation_site(k)`; `p_binding`, a `entry_site(e)`; `p_return`, a `operation_site(r)`. Tipo concreto desconhecido não impede transmissão precisa com prova. Remover uma prova impede a transmissão precisa correspondente. Prova limitada a `k` não autoriza `k2`, a entrada ou o corpo do chamado; `entry(e)` não cobre `r`. Preservar os sujeitos de assinatura instanciados por site e papéis chamador/chamado, inclusive sob repetição/recursão; referência exige também associação de local/vista apropriada.

**O-83-SCALAR:** O valor capturado chega ao parâmetro por `value`/`copy`; no corpo identidade do cenário, `return` o transmite ao resultado normal. Conteúdos de chamadas diferentes não são igualados por compartilhar assinatura/domínio. `reference` disponibiliza local sem afirmar cópia ou leitura de conteúdo pelo modo sozinho. Se o corpo devolver outro valor do mesmo domínio, esse valor rege o resultado.

**Falha a detectar:** Trocar transmissão comprovada por resultado arbitrário, igualar argumentos/resultados só por tipo, reutilizar prova fora do escopo ou inferir efeito do modo de passagem.

## O-84 — Cobertura de domínio em escolhas

**Cenário:** X-39, incluindo a premissa universal sobre a ocorrência inteira `pc` com restante aberto. Contracasos: prova só para um candidato; restante aberto sem garantia de domínio; premissa sobre outra ocorrência ou fora do site; premissa universal ligando candidato `known(int)` a destino `known(text)`; avaliações independentes de escolha heterogênea; destino alternativo.

**Resultado obrigatório:**

**O-84-STRUCT:** A cópia exige prova que cubra todas as combinações admissíveis e o restante; com essa prova, é válida apesar de `unknown_type`. Admitir a premissa `sameDomain(pc,@dst)` em `operation(c)` como garantia sobre todos os candidatos e membros do restante de `pc`, sem campo adicional de tipo do restante. Provas apenas dos candidatos não validam a escolha aberta. Rejeitar a contradição concreta, a referência a outra ocorrência como se fosse `pc` e o uso fora do site. Mesma representação de escolha heterogênea não prova domínio comum de duas avaliações. Preservar candidatos, restante, ocorrências, `TypeRef` e escopos das premissas; não exigir que o Validator certifique a verdade externa da garantia universal.

**O-84-SCALAR:** Conservar as alternativas de valor da origem; na variante de destino alternativo, a escrita permanece fraca em cada candidato. A prova de domínio não seleciona local, elimina restante ou prova disjunção.

**Falha a detectar:** Validar a partir do primeiro candidato, ignorar restante, confundir identidade de expressão com captura única ou promover escrita alternativa a obrigatória em todos os locais.

## O-85 — Escopos de prova fechados e decidíveis

**Cenário:** X-40. Duas premissas de domínio comum compõem a prova de uma cópia, com escopos de publicação, unidade, entrada, operação e interseções. Há duas chamadas distintas e uma unidade chamada. Variantes com escopo malformado, interseção vazia, unidades contidas e repetição/recursão.

**Resultado obrigatório:**

**O-85-STRUCT:** Interpretar exatamente os sites e formas de `DomainProofScope` definidos em 02, §1.4. Para `copy` de `U`, `publication ∩ operation(copy)`, `unit(U) ∩ operation(copy)` e suas composições finitas cobrem `operation_site(copy)`; `operation(other) ∩ operation(copy)`, `unit(V) ∩ operation(copy)` e `entry(eU) ∩ operation(copy)` são vazios. Só as primeiras variantes sustentam a cópia quando não há outra prova. `operation(k) ∩ invocation(k)` cobre a transmissão em `k`, não a avaliação local, outra chamada, entrada ou corpo do chamado. `unit(U)` cobre seus próprios sites, sem propagar-se a unidades contidas/chamadas; `publication` cobre todos os sites, sem dispensar a correspondência dos sujeitos. Rejeitar IDs inexistentes/de domínio incorreto, `invocation` de não-`invoke`, formas não admitidas e composição não finita, mesmo em premissa não utilizada. Preservar a quantificação universal sobre execuções e a distinção dos vínculos chamador/chamado, sem inferir alcançabilidade, igualdade de ativações ou um escopo dinâmico. Uma interseção vazia bem formada é admitida, mas seu uso como única prova viola I-08/I-52. Esses resultados não exigem calcular CFG, efeitos, RD ou valores.

**Falha a detectar:** Tratar escopo como texto livre, substituir interseção vazia por ancestral comum, estender escopo de entrada ao corpo ou de chamada ao chamado, reutilizar vínculo de outra chamada/ocorrência, aceitar `activation(...)` ou delegar aplicabilidade a uma análise de execução.

## O-86 — Contrato fechado sem inventário externo

**Cenário:** X-41; variante com `ContractRef` desconhecido e assinatura parcial; contracasos com origem inexistente, referência a posição não materializada e assinatura interna apontando para entrada diferente do target.

**O-86-STRUCT:** Preservar assinatura externa por `invoke`, posições, modos, `TypeRef`, limites, outcomes, autoridade/versão/evidência e premissas aplicáveis. Retirar acesso ao produtor/autoridade não altera os fatos disponíveis. Os contracasos violam I-02/I-55. Contrato conhecido com assinatura parcial permanece parcial; contrato desconhecido conserva fatos independentes e `CONTRACT_UNKNOWN`. Não admitir `ContractId`, inventário `contracts` ou lookup para completar conteúdo como formas do núcleo. Duas chamadas com mesma evidência mantêm sujeitos externos distintos; uma prova em `k` não valida `k2`.

**Falha a detectar:** Referência contratual “mágica”, aridade desconhecida convertida em zero, ID de implementação exigido para interpretar assinatura ou prova reutilizada por igualdade da autoridade.

## O-87 — Recurso declarado não é target de execução

**Cenário:** Um recurso declarado descreve o nome literal `billing`; dois sites usam esse nome com origens próprias. Variante interna aponta para uma entrada declarada sem corpo. Contracaso usa `ResourceId` no campo de target interno ou como quarta variante executável.

**O-87-STRUCT:** Conservar os dois sites e suas origens, nome/categoria/namespace/política e restante contratual; não inferir existência de artefato. Target interno fecha sobre `EntryId` e conserva assinatura/corpo indisponível. Contracaso viola I-57, mesmo que a declaração de recurso exista. Nome calculado referencia sua avaliação na operação; declaração de recurso não o captura nem cria execução. Relação de artefato não recebe avaliação dinâmica fictícia.

**Falha a detectar:** Resolver recurso declarado como entrada, fundir usos pela descrição compartilhada ou reavaliar uma expressão sem ponto de execução.

## O-88 — Disjunção como fato bilateral

**Cenário:** X-14 com `disjoint_storage({base(target),base(scratch)})`, autoridade e origem do cenário. Contracasos estruturais: base inexistente, repetida, conjunto com menos de duas bases ou escopo seletivo. Variante incompleta remove a premissa sem fornecer outra prova. Outra variante faz um destino aberto poder alcançar `target`, além de `scratch`.

**O-88-STRUCT:** Preservar a premissa par a par universal, seus sujeitos e evidência; rejeitar os contracasos por I-02/I-58. Remover a premissa não torna a publicação estruturalmente inválida por si só, mas remove a garantia de independência. Não transformar `ObjectId` em base nem transportar separação de uma escolha para seu restante. Verificar forma não certifica a verdade física da premissa.

**O-88-SCALAR:** Sob a garantia válida de X-14, a escrita limitada a `scratch` conserva `KNOWN` em `target`, como O-15. Sem prova ou com destino aberto que possa alcançar `target`, não afirmar o mesmo resultado fechado por IDs distintos.

**O-88-REGION:** Vistas não sobrepostas da mesma região podem ser separadas pelos intervalos existentes, sem exigir uma premissa entre bases distintas.

**Falha a detectar:** Fabricar independência para validar um exemplo, exigir premissa redundante para intervalos já disjuntos ou usar a premissa como garantia de tipo/codec.

## O-89 — Precondição sem token de Validator

**Cenário:** Recorte literal de texto dentro dos limites e sua variante fora dos limites; acesso puro cuja totalidade é sustentada pelo produtor; escolha aberta com premissa universal de O-84; igualdade de extensão cujo manifesto define ou não a operação. Um validador pode ter capacidade limitada para avaliar evidência não literal.

**O-89-STRUCT:** O recorte válido não exige `boundsProof` ou `SafetyAssertion`. Contradição literal de limites é rejeitada por I-09/I-46, mesmo acompanhada de token de “segurança”. Preservar a garantia de escolha pelo sujeito/escopo normativo, sem `knownRemainderDomainProof`. A igualdade de extensão depende do manifesto; token `EXTENSION_EQUALITY_DEFINED` não o substitui. As formas privadas de safety não são asserções do núcleo. Quando a verdade de uma precondição não puder ser verificada, registrar obrigação/limite explícito, sem declarar que a precondição foi provada ou inventar análise de valores.

**Falha a detectar:** Certificar semântica por enum de implementação, rejeitar forma válida apenas por ausência de certificado privado ou apagar precondição ao remover a classe.

## O-90 — Envelopes conservam a espécie de continuação

**Cenário:** `invoke` com retorno normal, exceção específica, catch-all, halt, divergência e restante; `opaque` com saltos locais e saída normal da unidade; `copy_bytes` comum com fallback que continua na operação seguinte. Contracasos duplicam normal/tag/catch-all de invocação, usam label de outra unidade, repetem chave de efeito ou colocam `continue` em terminador.

**O-90-STRUCT:** Preservar cada alternativa e restante sem fundir `InvocationOutcomes` com as permissões do envelope genérico. Rejeitar os contracasos por I-02/I-60. `continue` do fallback comum não exige nova sequência nem constitui segunda execução. `return` do envelope sai da unidade; `normal(label)` conserva destino local. O limite `otherwise` cobre outcomes sem limite específico e o restante aberto; ausência de informação não vira `none`. Este oráculo verifica fatos/regras de controle e não calcula CFG, efeitos ou valores.

**Falha a detectar:** Introduzir fallthrough em `opaque`, tratar saída da unidade como label normal ou perder fallback de operação comum por só saber representar destinos em labels.

## O-91 — Identidade e contrato abstrato independem de implementação

**Cenário:** Relações de artefato distintas, operandos em operações e em condições de entrada, e um `return` compartilhado por múltiplas entradas. Renomear detalhes privados de uma implementação e remover seus certificados não muda os fatos AIR. Variante troca o domínio/proprietário de uma referência de relação/operando.

**O-91-STRUCT:** Identificar relações por `ArtifactRelationId`, fechar proprietários e rejeitar a referência trocada por I-01/I-02/I-11. `ContractRef` não adquire ID por internamento em memória. O retorno conserva valores e a regra da ativação corrente, sem seletor `entryScope`; limite em validar compatibilidade de múltiplas entradas é registrado, sem eliminar execuções. Mudança de classes, representação de números ou transporte não redefine validade. Inteiros não recebem limite de um runtime pelo modelo abstrato.

**Falha a detectar:** Exigir identidade Java para fechar fato, misturar ocorrências pelo objeto lido, selecionar entradas para fazer a validação passar ou alterar semântica pela codificação.
