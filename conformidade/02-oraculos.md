# Oráculos de conformidade

**Analysis IR 2.0.0 — Normativo**

## Convenções

Cada oráculo define comportamento de conformidade; não prescreve um framework ou uma implementação. Os exemplos associados são encontrados em [fluxo](../exemplos/01-fluxo-e-valores.md), [memória](../exemplos/02-memoria-e-chamadas.md), [extensões](../exemplos/03-extensoes-e-parcialidade.md) e [conhecimento de tipo](../exemplos/04-conhecimento-de-tipo.md).

Quando se exige um conjunto exato, valem as premissas do cenário, o perfil preciso pertinente e o escopo de entradas explicitado. Consumers apenas conservadores podem produzir resultados mais amplos com limites declarados, mas não satisfazem por isso o perfil de precisão que exige aquele resultado. Os valores textuais exibidos sem aspas nesta tabela continuam literais de texto; IDs RD designam eventos, não valores.

Para invariância de identidade entre representações equivalentes, compara-se por uma correlação explícita de IDs, e não por igualdade de números locais.

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

**Resultado obrigatório:** Com contrato especializado aplicável, a fatia e a regra de padding permitem target PGMA; sem contrato, só ROUTER e os valores do argumento são sustentados.

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

**Resultado obrigatório:** Objeto, célula, `object(@name)`, `read` e ocorrência de leitura conservam `known(text)`. A atribuição satisfaz I-08; o literal A é sustentado após a escrita.

**Falha a detectar:** Perder o domínio ao propagá-lo entre declaração, local e operando, ou exigir frontend para identificá-lo.

## O-70 — Domínio de extensão não é tipo desconhecido

**Cenário:** X-31, objeto `@token` com `known(opaque_type(example.token,1))`, inclusive consumidor sem interpretação interna do domínio. Variante: cópia entre duas células desse mesmo domínio sob contrato de consumo conservador; contracaso: `concat(read(@token),text("A"))`.

**Resultado obrigatório:** O domínio e a versão permanecem identificados; `havoc`, leitura e cópia de valor entre células compatíveis preservam esse domínio. A concatenação é `INVALID_IR`. Operações precisas de extensão exigem contrato/capacidade; ausência de suporte não vira `TYPE_UNKNOWN`.

**Falha a detectar:** Tratar domínio opaco como tipo ausente, aceitar texto implicitamente, inventar igualdade não definida ou usar `opaque_type("unknown",1)` para uma lacuna de domínio.

## O-71 — Entidade com domínio não estabelecido

**Cenário:** X-31, objeto `@untyped` com `unknown_type(u)` e célula identificada. Variantes negativas: remover a lacuna `u` ou alterar seu código para um que não seja `TYPE_UNKNOWN`.

**Resultado obrigatório:** O objeto e a célula permanecem presentes com origem e associação conhecidas; tipo desconhecido não implica storage/binding desconhecidos. As variantes negativas são `INVALID_IR` por I-02/I-49.

**Falha a detectar:** Apagar declaração, fabricar célula por causa do tipo, perder a lacuna ou aceitar referência de tipo sem significado.

## O-72 — Leitura de tipo desconhecido em operação neutra

**Cenário:** X-32. Variante metamórfica: duas ocorrências de `read(@x)` em `knownOperands`, cada uma referenciada no envelope.

**Resultado obrigatório:** Conservam-se `ObjectId`, `StorageId`, `TypeRef`, lacuna de tipo, razões aplicáveis ao conteúdo, origem da declaração e origem/identidade de cada ocorrência. A operação tem as leituras conhecidas e continuação declarada, sem escrita. Referir uma ocorrência no envelope não duplica o evento; duas ocorrências distintas não são fundidas.

**Falha a detectar:** Substituir por `nop`, apagar a leitura, inventar escrita/recurso ou perder a distinção entre uso e declaração.

## O-73 — Tipo desconhecido em aritmética

**Cenário:** X-34, `add(read(@x),int(1))`; variantes `neg(read(@x))` e ambos argumentos de tipo desconhecido com a mesma lacuna.

**Resultado obrigatório:** `INVALID_IR` por I-08. O domínio não é inferido do operador, do outro argumento ou do ID da lacuna. `add(unknown(known(int)),int(1))` é válido e mantém resultado de domínio `int` não determinado.

**Falha a detectar:** Interpretar “tipo desconhecido” como dispensa de validação, assumir inteiro ou considerar as variantes positivas como tipo desconhecido.

## O-74 — Tipo desconhecido em texto e booleanos

**Cenário:** X-34, `concat(read(@x),text("A"))`, `not(read(@x))` e variante `branch read(@x)`.

**Resultado obrigatório:** Cada contracaso é `INVALID_IR` por I-08. A variante booleana de X-33 é válida porque seu resultado é explicitamente `known(bool)` e tem pureza estabelecida; conserva a dependência de tipo desconhecido e ambos os destinos.

**Falha a detectar:** Assumir `text`/`bool`, converter silenciosamente a dependência ou confundir desconhecimento do predicado com desconhecimento de seu tipo.

## O-75 — Valor desconhecido de domínio conhecido

**Cenário:** X-33, `p = unknown(known(text),[],none,v)`; variante de atribuição a célula `known(text)` em X-34.

**Resultado obrigatório:** O domínio é `text`, o valor não é determinado e a razão de valor permanece. A atribuição é válida. Avaliações distintas não têm igualdade garantida. Não é necessário criar lacuna `TYPE_UNKNOWN`.

**Falha a detectar:** Transformar `unknown` em literal, apagar a expressão, abrir o domínio sem motivo ou rejeitar uma operação apenas pelo valor desconhecido.

## O-76 — Valor e domínio desconhecidos

**Cenário:** X-33, `q = unknown(unknown_type(u),[read(@x)],none,v)`.

**Resultado obrigatório:** As lacunas de tipo e valor são distinguíveis; a dependência, sua ocorrência e sua origem permanecem. `remainingReads=none` fecha a lista de leituras, não o domínio ou o conjunto de valores. Não se cria igualdade entre domínio/valor do resultado e da dependência.

**Falha a detectar:** Substituir a referência de tipo pela razão de valor, perder dependências, inventar um domínio padrão ou interpretar ausência de candidatos como conjunto vazio fechado em ponto alcançável.

## O-77 — Assign exige compatibilidade; abstração preserva a leitura

**Cenário:** X-34, tentativas de `assign` com tipo desconhecido em um ou nos dois lados; variante `assign(@x,read(@x))`; abstração `transfer` com envelope explícito.

**Resultado obrigatório:** As atribuições precisas são `INVALID_IR` por I-08, mesmo com o mesmo objeto/ID de lacuna. A abstração é válida e mantém leitura da origem, ocorrência de escrita e sobrescrita completa comprovada do destino. Não afirma cópia tipada exata. `assign` entre domínios conhecidos diferentes também continua inválido.

**Falha a detectar:** Criar compatibilidade universal/implícita, inferir tipo por igualdade de lacunas ou substituir a abstração apenas por `havoc` apagando a leitura.

## O-78 — Choice e binding preservam tipos próprios

**Cenário:** X-35. Variantes: escolha fechada só de locais `known(text)`; escolha vazia fechada; candidato com tipo desconhecido; binding físico `unknown(scope,reason)` em objeto de tipo `known(text)`.

**Resultado obrigatório:** A escolha homogênea fechada conserva `known(text)`; as heterogêneas ou sem garantia para o restante usam `unknown_type` preservando candidatos e seus domínios. `known(text)` sem essa garantia viola I-51; escolha vazia fechada é inválida para leitura/escrita. Binding físico desconhecido conserva `known(text)` do objeto.

**Falha a detectar:** Escolher o primeiro candidato, tratar desconhecimento como conversão/união de tipos, perder tipos conhecidos ou misturar lacuna nominal/física com lacuna de domínio.

## O-79 — Assinaturas parciais e precondições concretas

**Cenário:** X-36 e seus contracasos. Variante positiva: parâmetro, argumento e resultado de mesmo domínio conhecido.

**Resultado obrigatório:** A chamada parcial conserva posições, modos, `TypeRef`, leituras, alvo e outcomes conhecidos, sem alegar transmissão tipada precisa. Resultado desconhecido é escrito só no retorno normal. Argumento de tipo desconhecido para parâmetro `known(int)` e `return` de tipo desconhecido para resultado `known(text)` violam I-08; domínios conhecidos compatíveis satisfazem a transmissão precisa. Lacuna de tipo de uma posição não apaga aridade nem torna assinatura desconhecida uma assinatura vazia.

**Falha a detectar:** Aceitar precondição concreta sem prova, tratar posição desconhecida como polimorfismo, fabricar argumento/resultado ou aplicar resultado normal na exceção.

## O-80 — Havoc não muda o conhecimento do domínio

**Cenário:** Célula exata `@x : unknown_type(u)` com conteúdo de entrada desconhecido; executar `havoc.must @x`. Variante independente: `havoc.may {@x}`. Demais locais são comprovadamente disjuntos.

**Resultado obrigatório:** Ambas preservam `unknown_type(u)` e a razão de escrita. A primeira sobrescreve obrigatoriamente a célula; a segunda admite ausência de escrita. A lacuna de tipo não muda extensão, identidade ou independência já estabelecidas nem contamina locais disjuntos.

**Falha a detectar:** Escolher domínio novo, converter o conteúdo, apagar escrita comprovada ou abrir todos os efeitos apenas por desconhecer o tipo.

## O-81 — Domínio lógico e codec são conhecimentos distintos

**Cenário:** Três variantes com mesmo intervalo físico válido e conhecido: vista com `text.ascii@1`; vista de domínio `known(text)` mas codec não interpretado; vista cujo domínio lógico e interpretação não são estabelecidos, com `unknown_type(u)`. Nas variantes incompletas, o cenário garante leitura total sem efeitos excepcionais.

**Resultado obrigatório:** A primeira tem `known(text)`; a segunda conserva `known(text)` e `CODEC_UNKNOWN`; a terceira conserva `unknown_type(u)` e lacuna de interpretação. Intervalo, bytes e leitura conhecidos não desaparecem. Cópia bruta continua possível sob as precondições de `copy_bytes`. Se totalidade não estiver assegurada, a leitura exige operação com envelope de erro/controle apropriado. `assign` preciso sem domínio/codificação aplicáveis não é autorizado.

**Falha a detectar:** Fazer codec desconhecido implicar sempre tipo desconhecido, escolher ASCII implicitamente, alegar `unknown_type` apesar de codec que estabelece domínio conhecido ou apagar fatos físicos independentes.
