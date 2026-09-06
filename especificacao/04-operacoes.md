# 04 — Catálogo de operações

**Analysis IR 2.0.0 — Normativo**

## 1. Cabeçalho comum

Toda operação DEVE possuir `OperationId`, espécie semântica versionada, proprietário, operandos identificados e ordenados quando a ordem for significativa, origem, cobertura e precisão por dimensão. Incertezas são referidas por identidade. O terminador também é uma operação e participa do mesmo inventário.

Operações comuns prosseguem para a próxima operação da mesma sequência. Terminadores seguem sua própria regra. Não há resultado implícito de controle na ausência de terminador.

Os parâmetros `T: Type` e `R: TypeRef` seguem [02 — Tipos](02-tipos-valores-e-operandos.md). Tipos concretos nas assinaturas exigem `known(T)`; tipo desconhecido nunca dispensa uma precondição. Quando um operando denota valor/local, seu `TypeRef` e lacunas DEVEM ser preservados também em contratos e envelopes.

## 2. `assign`

```text
assign(destination: Place<Rdst>, value: Expression<Rsrc>)
requires sameDomain(destination, value)
```

Avalia destino e valor no estado anterior, grava o valor no destino e continua. A escrita é completa para o local designado. Não realiza conversão implícita, concatenação, truncamento ou preenchimento. As leituras incluem todas as leituras da expressão e todos os cálculos de endereço do destino.

`Rdst` e `Rsrc` são `TypeRef` independentes; destino e expressão DEVEM satisfazer `sameDomain` conforme [02, §1.3](02-tipos-valores-e-operandos.md#13-igualdade-de-domínios-sem-identificação-concreta). `known(T)` nos dois lados fornece uma prova, inclusive para `opaque_type(id, version)`, mas identificar `T` não é obrigatório para copiar um valor. `assign(object(x),read(object(x)))` sobre célula é válido por identidade; alias exato e premissa de mesmo domínio também podem sustentar a cópia. Compartilhar apenas `UncertaintyId` entre objetos distintos não basta.

Uma cópia válida conserva a relação entre o valor gravado e a avaliação da origem no estado anterior, mesmo quando o domínio e o valor concreto permanecem desconhecidos. A ocorrência de leitura e o evento de definição continuam presentes, inclusive na autoatribuição. `sameDomain` não dispensa tamanho, codificação, limites ou ausência de conversão exigidos pela memória e pela operação. Se esses fatos e a cópia de identidade forem conhecidos, desconhecer a identidade concreta do domínio não justifica substituí-la por escrita arbitrária.

Uma atribuição observada sem prova aplicável de compatibilidade DEVE ser preservada por `opaque`, com leitura da origem, destino e escrita obrigatória/possível conforme os fatos conhecidos. `havoc` sozinho não substitui essa leitura conhecida. Conhecer só o domínio comum também não prova que uma construção observada efetua cópia sem conversão; essa semântica precisa estar estabelecida antes de usar `assign`.

Destino exato permite atualização forte segundo as condições de memória. Destino alternativo representa uma escrita em uma das alternativas admitidas, não em todas; o consumidor aplica atualização fraca às alternativas. Uma alternativa aberta inclui o escopo do restante. A escolha não é resolvida pela ordem de apresentação.

Um único `assign` não produz valor temporário reutilizável. Para capturar um valor, o destino pode ser uma célula auxiliar com proveniência derivada. Múltiplas atribuições sequenciais não podem substituir atualização simultânea quando houver dependências entre seus operandos.

## 3. `havoc`

```text
havoc.must(destination: Place<R>, reason)
havoc.may(scope: MemoryScope, reason)
```

`havoc.must` grava um valor desconhecido no local, obrigatoriamente. Para destino exato, elimina valores anteriores do local completamente sobrescrito. Para `choice`, obriga escrita em uma alternativa, não em todas.

`havoc.may` pode escrever valores arbitrários em qualquer subconjunto do escopo, inclusive o conjunto vazio. Acrescenta definições possíveis e restante desconhecido, sem eliminar definições que podem sobreviver. Ambos preservam a continuação sequencial e NÃO modelam controle desconhecido. Se controle também for desconhecido, utiliza-se `opaque`.

Ambos conservam o `TypeRef` de cada local. Com `unknown_type`, o conteúdo passa a ser desconhecido em um domínio ainda não estabelecido; não se escolhe um domínio novo nem se converte um valor. A falta de tipo, isoladamente, não enfraquece uma sobrescrita completa comprovada. As garantias de local, extensão da escrita e controle continuam necessárias.

Uma entrada de ambiente que certamente atribui ao destino pode usar `havoc.must`; seus resultados são desconhecidos, mas o fato da escrita não é.

## 4. `nop`

```text
nop()
```

Não lê nem escreve armazenamento e prossegue. Só pode representar ausência de efeito comprovada. Uma operação não compreendida, um trecho ausente ou uma extensão ignorada NÃO DEVE ser reduzido a `nop`.

## 5. `copy_bytes` — `memory.regions@1`

```text
copy_bytes(destination: ByteRange, source: ByteRange, length: nonnegative int)
```

Captura exatamente `length` bytes do intervalo de origem no estado anterior e grava essa imagem no destino. Os intervalos devem estar dentro dos limites admissíveis. Sob sobreposição, a semântica é de captura anterior à atualização, nunca de leitura progressivamente modificada pela própria cópia.

Não converte representações. Leituras e escritas são intervalos distintos, mesmo quando pertencem à mesma região. `length=0` não altera memória; os cálculos de endereço puros permanecem identificáveis. Usar essa operação exige que a captura corresponda à semântica que o produtor pretende representar.

## 6. Transferências diretas

```text
jump(destination: LabelId)
branch(predicate: Expression<bool>, trueDestination: LabelId, falseDestination: LabelId)
dispatch(selector: Expression<T>, cases: [(Literal<T>, LabelId)], default: LabelId)
```

São terminadores. `jump` transfere incondicionalmente. `branch` avalia seu predicado e escolhe um destino; predicado desconhecido preserva os dois como possibilidades. Destinos iguais são válidos e não apagam a avaliação do predicado.

Em `branch`, desconhecido significa valor desconhecido com `known(bool)`. `unknown_type` não satisfaz a assinatura. `dispatch` também exige domínio conhecido do seletor e dos literais.

`dispatch` avalia o seletor uma vez, compara por igualdade de tipo e escolhe a única entrada correspondente, ou `default`. O domínio `T` é `bool`, `int`, `decimal`, `text` ou `bytes`. Literais duplicados por igualdade semântica são inválidos. A ordem da tabela não estabelece prioridade porque os casos são disjuntos. `default` é sempre explícito, inclusive quando representará término ou erro em sequência própria.

Uma seleção por predicados sobrepostos com prioridade não é `dispatch` direto: deve ser reduzida a testes ordenados ou extensão própria.

## 7. `invoke`

```text
invoke(
  action: InteractionAction,
  target: InternalEntry | LiteralResource | ComputedResource,
  arguments: Argument[],
  results: Place[],
  effectBound: ForeignEffectBound,
  outcomes: InvocationOutcomes,
  contract: ContractRef | unknown
)
```

`invoke` é terminador e modela uma interação externa à sequência corrente. As ações padrão são `call`, `open`, `close`, `read`, `write`, `update`, `delete` e `execute`. A ação e a categoria do recurso permanecem distintas: `read` de `table` não vira chamada a `program`.

O alvo interno identifica uma entrada disponível ou explicitamente declarada sem corpo. O alvo literal identifica categoria, namespace, texto do nome e política de interpretação. O alvo calculado identifica os mesmos componentes, mas seu nome vem de expressão `text`. Um alvo originalmente em bytes deve usar decodificação explícita ou permanecer desconhecido. Não há remoção implícita de espaços ou canonicalização por caixa.

Um alvo calculado exige `known(text)`. Se só o resultado textual da interpretação estiver estabelecido, pode usar `unknown(known(text), dependencies, remainingReads, reason)` conservando as leituras originais, inclusive as de tipo desconhecido. Isso é uma abstração explícita da interpretação do nome, não uma coerção do operando. Se nem esse contrato estiver estabelecido, a interação permanece em `opaque` com seu envelope de dependências.

Target e argumentos são avaliados no estado anterior, antes de efeitos da interação. Um conjunto de valores de target é resultado de análise posterior; não é campo exigido da operação. Target interno por identidade não exige resolver novamente seu nome.

### 7.1 Argumentos

| Modo | Significado |
| --- | --- |
| `value(expr)` | Captura o valor antes da interação |
| `reference(place)` | Disponibiliza o local e calcula seu endereço antes da interação |
| `copy(expr)` | Captura valor para armazenamento privado da interação; alterações nessa cópia não atualizam o argumento original |

Modos NÃO garantem pureza. `value` e `copy` não protegem armazenamento compartilhado acessível por outra via. `reference` não prova leitura ou escrita do conteúdo: isso depende do contrato ou corpo chamado. Sem essa informação, os efeitos sobre o local referido são conservadores.

A ordem da lista identifica a posição de cada argumento. Nenhum argumento omitido é fabricado. Assinatura parcial usa incerteza explícita e efeito apropriado. Destinos de resultados normais são escritos após o retorno normal; antes da interação, só seus endereços são avaliados. O resultado devolvido é desconhecido na ausência de corpo/contrato que o restrinja.

Cada posição de assinatura usa `TypeRef`. `value`/`copy` conservam o `TypeRef` da expressão capturada; `reference` conserva o do local, sem convertê-lo em leitura de conteúdo. A transmissão sem conversão exige `sameDomain` entre argumento e parâmetro, entre valor de `return` e posição de resultado da entrada e entre resultado devolvido e destino no chamador. A associação precisa de parâmetro por referência exige a mesma prova de domínio/vista e suas condições de armazenamento. As posições podem ter domínio concreto desconhecido com prova por identidade ou contrato explícito; uma lacuna compartilhada sozinha não é assinatura polimórfica.

As provas identificam posições, ocorrências e vínculos chamador/chamado, com `DomainProofScope` conforme [02, §1.4](02-tipos-valores-e-operandos.md#14-escopo-de-provas-de-domínio). A transmissão é verificada em `invocation_site(k)`; o vínculo declarado na entrada, em `entry_site(e)`; a compatibilidade do valor devolvido por `return r`, em `operation_site(r)`. O escopo de uma chamada não se estende ao corpo chamado. Um contrato pode declarar relações entre parâmetro e resultado e seu vínculo com objetos da entrada; sua aplicação à chamada segue esses vínculos, sem unir ativações distintas por IDs locais. Igualdade de domínio entre parâmetro e resultado não afirma que o resultado copie o argumento: essa relação de valor depende do corpo ou de contrato de valor explícito. `value`, `copy`, inicialização por parâmetro e `return` conservam a relação de captura que suas próprias semânticas estabelecem. Precondições adicionais que interpretem um domínio concreto continuam exigindo `known(T)` e as demais garantias aplicáveis.

Uma assinatura parcial pode conservar posições `unknown_type(u)` e argumentos de tipo desconhecido em `invoke` com contrato conservador quando não houver prova de transmissão precisa. Isso registra a interação e as capturas/locais conhecidos, sem afirmar compatibilidade de domínios ou vinculação precisa entre argumento e parâmetro. Resultados sem relação de transmissão estabelecida conservam o `TypeRef` do destino e recebem conteúdo desconhecido apenas no retorno normal, se essa escrita estiver assegurada. Se houver `sameDomain` e transmissão sem conversão estabelecidos, as relações de cópia DEVEM ser preservadas mesmo com tipo concreto desconhecido; não se exige o fallback apenas pela lacuna. Cardinalidade, modos, efeitos e outcomes conhecidos continuam obrigatórios; os desconhecidos requerem suas próprias lacunas. Contradição de domínios, precondição concreta não satisfeita ou regra de resultados não assegurada exige `opaque` com envelopes apropriados, não aceitação de transmissão precisa violando o contrato.

A tupla de valores retornados é capturada antes das atribuições de resultados. Os destinos são atualizados na ordem declarada em `results`; se houver aliases, a escrita posterior pode sobrescrever a anterior. Uma linguagem com ordem diferente deve ser normalizada usando resultados auxiliares e atribuições explícitas. Sem alternativa de retorno normal, a lista `results` DEVE ser vazia.

### 7.2 Limites de efeitos externos

`ForeignEffectBound` declara limites superiores de leituras e escritas sobre armazenamento do chamador/compartilhado, por resultado quando houver distinção. `none` é garantia explícita de ausência desses efeitos; `may_read(S)` e `may_write(S)` admitem qualquer subconjunto do escopo. Um limite omitido ou desconhecido equivale ao escopo potencialmente visível mais amplo, não a `none`.

Um limite pode indicar sobrescrita completa obrigatória de local exato quando uma autoridade semântica o garante. Essa obrigação não pode ser inferida apenas do modo de passagem ou nome da rotina. Escritas dos destinos de resultados normais são adicionais aos efeitos do corpo.

Esses limites são premissas contratuais, não um resumo de análise calculado disfarçado de IR. Um resumo produzido posteriormente é produto separado. Se um corpo disponível contradiz um limite, a composição é inconsistente; o consumidor deve rejeitar a alegação ou usar um limite conservador explicitamente justificado.

### 7.3 Resultados de controle

`InvocationOutcomes` contém até um destino normal, destinos excepcionais por tag, destino opcional para qualquer outra exceção, possibilidade de término do processo, possibilidade de divergência e restante aberto quando necessário. Seu significado completo está em [05 — Controle](05-controle-e-invocacoes.md).

Um destino normal significa que a chamada **pode** retornar ali; não promete que sempre retorna. Ausência comprovada de retorno normal é diferente de comportamento desconhecido. Se houver resultados normais, `results` deve corresponder à assinatura; na saída excepcional não há escrita implícita desses resultados.

## 8. Saídas

```text
return(values: Expression[])
raise(tag, values: Expression[])
halt(kind: normal | abnormal)
```

`return` retorna da ativação da unidade corrente ao invocador, após avaliar os valores. Em entrada raiz, termina essa invocação raiz normalmente. A assinatura precisa ser compatível.

`raise` encerra a ativação com saída excepcional. O destino vem da interação invocadora ou é uma saída excepcional raiz. `halt` termina a execução analisada; não equivale a retorno de subrotina. Nenhuma dessas operações possui fallthrough local.

A V2 não define comportamento indefinido como licença para apagar análise. Um comportamento-fonte inválido ou não modelado deve conservar diagnóstico e envelope adequado.

## 9. `opaque`

```text
opaque(
  observedKind: QualifiedName,
  knownOperands: Operand[],
  valueResults: Place[],
  memoryEnvelope: MemoryEnvelope,
  controlEnvelope: ControlEnvelope,
  dependencyEnvelope: DependencyEnvelope,
  reasons: UncertaintyId[]
)
```

É um terminador que conserva uma construção observada cuja semântica não é totalmente interpretada. Pode ter continuação normal conhecida, mas isso deve ser sustentado pelo envelope, não pela sua posição textual.

Os operandos conhecidos são preservados e seus papéis mantidos. `valueResults` só designa escritas se o envelope o declarar; ausência de resultados conhecidos não prova ausência de escrita. Os três envelopes têm definição em [06 — Incompletude](06-incompletude-e-proveniencia.md).

`knownOperands` pode conter `read` com `unknown_type`, sem exigir interpretação desse domínio. Sua ocorrência, origem, lacuna de tipo e dependências conhecidas permanecem observáveis no ponto da operação. Um envelope com essa leitura, sem outras leituras/escritas, continuação normal conhecida e sem outros usos de recurso é admissível quando esses fatos forem estabelecidos. Não equivale a `nop`, que perderia a leitura. Uma anotação `TYPE_UNKNOWN` não permite ocultar efeitos ou inventar limites vazios.

Uma operação opaca não precisa apagar os fatos anteriores ou adjacentes. Entretanto, qualquer análise que atravesse sua região de influência deve incluir seus efeitos e seu restante desconhecido. Payload de exibição ou nome de construção não pode ser reinterpretado pelo consumidor como substituto de semântica.

## 10. Operações de extensões padronizadas

`local.invoke`, `local.boundary`, `local.resume`, `local.unwind` e `indirect.jump` são definidas integralmente em [05](05-controle-e-invocacoes.md). Não pertencem à exigência de precisão do perfil escalar. Toda extensão também deve possuir envelope conservador interpretável pelo núcleo.

## 11. Regra de ausência

Uma coleção vazia de argumentos, resultados, leituras, escritas ou destinos só significa conjunto vazio quando seu respectivo contrato estiver disponível e fechado. Ausência de contrato exige marcação própria. Nenhum accessor vazio ou campo omitido pode acumular os dois significados.
