# 07 — Contrato de produtores e lowering

**Analysis IR 2.0.0 — Normativo**

## 1. Independência

O contrato de entrada de um produtor pertence a ele; o contrato de saída pertence à IR. A conformidade NÃO exige que a entrada seja um produto de nome específico, uma AST ou uma representação orientada a objetos. O consumidor deve observar apenas a publicação IR e contratos externos explicitamente referidos por ela.

Um produtor de outra linguagem pode gerar a mesma operação com origem diferente. Não precisa reproduzir a taxonomia, os IDs, os namespaces nominais ou as fases de outro frontend. Nenhuma regra V2 depende de nome de linguagem ou palavra-chave-fonte.

## 2. Informações mínimas por capacidade

| Capacidade de saída | Fatos exigidos da entrada ou de uma regra de tradução válida |
| --- | --- |
| Declaração nominal | Identidade, `TypeRef` conhecido ou `unknown_type` com lacuna, origem e escopo |
| Célula independente | Identidade de armazenamento e justificativa de separação/alias |
| Atribuição | Destino, valor, regra de conversão e ordem de avaliação |
| Bifurcação | Predicado puro ou sua abstração, destinos de cada resultado e continuidades |
| Transferência | Destino(s) e natureza da transferência; ausência de retorno quando pertinente |
| Interação | Categoria/ação, forma do alvo, argumentos, resultados, controle e limites de efeitos |
| Acesso por região | Base, intervalo ou domínio de intervalos, unidades de medida e codec |
| Invocação local | Entrada, conclusão, continuação e disciplina de contextos |
| Construção não suportada | Identidade/origem observada, operandos conhecidos e limites conservadores |

Se um fato exigido não estiver disponível, o produtor DEVE reduzir a alegação ou usar representação opaca. Não deve preencher o campo com um default de semântica mais forte.

Tipo desconhecido não remove declaração, célula, leitura ou ocorrência conhecida. O produtor usa `unknown_type(u)` uniformemente e preserva fatos independentes. Não pode escolher `int`, `text`, `bool` ou `opaque_type` para satisfazer a assinatura desejada. Uma operação de domínio conhecido só é publicada se suas precondições forem satisfeitas; caso contrário, a construção e suas leituras/escritas conhecidas permanecem por abstração explícita.

Quando conhece uma cópia sem conversão, o produtor DEVE preservar essa relação de valor por `assign` ou transmissão pertinente se puder demonstrar `sameDomain`, ainda que não identifique o domínio concreto. Pode publicar uma premissa tipada de mesmo domínio, com autoridade, sujeitos, origem e `DomainProofScope` conforme 02, §1.4; não pode inferi-la apenas da operação que pretende validar. Falta de prova de compatibilidade e falta de identidade concreta do domínio são lacunas diferentes. Uma premissa limitada a um site não é generalizada a todas as chamadas; uma garantia apenas de uma ativação particular não sustenta um escopo estático universal. Para escolha aberta, a premissa sobre a ocorrência inteira exige evidência que cubra também todo o restante, não só os candidatos listados.

## 3. O que o lowering faz

O lowering converte fatos semanticamente estabelecidos em operações normalizadas. Pode decompor seleção, repetição, conversão ou atualização em operações mais simples, criar labels e objetos auxiliares, resolver continuações estruturais e copiar proveniência.

Essa transformação não é nova resolução nominal nem análise de valores. Traduzir uma atribuição seguida por bifurcação é permitido. Concluir qual atribuição alcança uma chamada, qual ramo será tomado ou qual programa será carregado pertence às análises posteriores.

A análise necessária para compreender semântica ainda ausente da linguagem deve ocorrer no produtor competente antes da tradução ou permanecer lacuna. O lowering NÃO DEVE interpretar texto de exibição para recuperar operadores, destinatários, faixas, convenções de retorno ou aliases que seu contrato de entrada não estabeleceu.

## 4. Obrigações de preservação

**PROD-01 — Comportamento.** Preservar o conjunto de comportamentos por inclusão conservadora, conforme [00](00-escopo-e-convencoes.md).

**PROD-02 — Inventário.** Representar todas as ocorrências da capacidade alegada. Não há seleção automática de primeiro/último uso ou pareamento obrigatório entre uma definição e uma chamada.

**PROD-03 — Ordenação.** Preservar avaliação, atualizações e transferências. Se uma expressão é capturada uma vez, não duplicar sua avaliação através de efeitos.

**PROD-04 — Identidade.** Preservar relações por identidades IR próprias e fechadas; nomes são apresentação ou nomes de recursos externos, não joins internos.

**PROD-05 — Armazenamento.** Não converter desconhecimento de layout em objetos artificialmente independentes. Incluir fatos declarativos suficientes para o perfil preciso, ou escopo de alias aberto.

**PROD-06 — Conversão.** Tornar preenchimento, truncamento, codificação, arredondamento e limites observáveis. Não declarar `assign` de identidade se a origem realiza conversão diferente.

**PROD-07 — Controle.** Não converter construção desconhecida em continuação sequencial. Não confundir salto, retorno local, retorno de unidade e término da execução.

**PROD-08 — Interações.** Preservar todos os argumentos, resultados e caminhos excepcionais cobertos. Alvo calculado continua expressão; nome literal continua observação nominal.

**PROD-09 — Incompletude.** Conservar gaps, escopos, dimensões e premissas. Fatos independentes continuam observáveis, mas não se afirma completude global na presença de inventário incompleto.

**PROD-10 — Proveniência.** Toda decomposição deve ser explicável. Origem derivada não se apresenta como trecho escrito. Clones semânticos possuem IDs distintos e relação com a origem comum.

## 5. Condições e avaliação contextual

Quando o significado de um predicado depende de informação de declaração ou contexto, essa interpretação deve estar estabelecida antes de usar uma expressão precisa da IR. A IR não herda abreviações ou categorias gramaticais implícitas.

Se apenas o esqueleto de controle e as leituras forem conhecidos, um predicado puro desconhecido é admissível e preserva ambos os caminhos. Se pureza, exceções ou efeitos também forem desconhecidos, é necessário `opaque` ou outra sequência que conserve esses comportamentos.

O produtor não precisa preservar sintaxe equivalente quando ela não muda semântica. Precisa preservar o que distingue efeitos, avaliação, destinos, tipos e origem. Um ramo vazio pode ser representado pelo mesmo destino da continuação, sem campo sintático de “cláusula presente”.

## 6. Compilação, contexto e ambiente

Fatos dependentes de ambiente devem carregar premissa ou estado desconhecido. Não ter uma configuração não invalida automaticamente fatos independentes, nem autoriza presumir uma configuração usual.

Políticas que mudam nomes externos, layout ou convenções de interação devem ser traduzidas para fatos normalizados e/ou identificadores versionados de contrato. O consumidor não interpreta flags de um compilador específico. Quando uma política não foi resolvida, apenas os fatos dependentes devem ser abertos, respeitada sua influência causal.

## 7. Publicação

O produtor DEVE validar fechamento, integridade, `TypeRef`, precondições de domínio das operações, inventários e referências antes de publicar. `unknown_type` não dispensa essas verificações. Deve rejeitar IR internamente inconsistente; recuperação de input parcial não justifica referências quebradas.

Uma publicação não pode depender de callbacks para completar fatos ao ser consultada. Descartar o produtor após publicar não altera a semântica disponível. Uma publicação produzida sem linguagem-fonte, com os mesmos fatos normalizados, deve ser igualmente consumível.

## 8. Critério de suficiência

Para afirmar que uma capacidade está disponível, deve ser possível entregar a publicação a um consumidor independente e reconstruir o comportamento/observações do seu oráculo sem acessar a entrada do produtor.

Preservar somente nomes e linhas não satisfaz esse critério. Preservar operações precisas onde possíveis e limites explícitos onde não possíveis satisfaz conformidade conservadora, mas não os oráculos de precisão dos perfis que dependem das informações ausentes.
