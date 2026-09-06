# 09 — Extensibilidade e compatibilidade

**Analysis IR 2.0.0 — Normativo**

## 1. Unidade de extensão

A extensão acrescenta uma capacidade semântica tipada e versionada. Não acrescenta campos singleton ao envelope fundamental para cada construção-fonte. Pode definir operações, tipos, codecs, categorias de recurso ou relações, desde que mantenha as fronteiras da IR.

Um identificador de extensão é qualificado por autoridade e nome. Identificadores `core.*`, `memory.regions`, `control.local` e `control.indirect` são reservados a esta especificação. A publicação identifica versão requerida de cada capacidade efetivamente usada.

## 2. Manifesto semântico obrigatório

Cada extensão DEVE definir:

| Elemento | Obrigação |
| --- | --- |
| Identidade e versão | Namespace não ambíguo e regras de compatibilidade |
| Motivação bilateral | Fato do produtor e observação exigida pelo consumidor |
| Sintaxe abstrata | Operand kinds, tipos, cardinalidades e atributos semânticos |
| Semântica | Avaliação, efeitos, controle, erros e dependências observáveis |
| Validade | Invariantes e condições de uso |
| Envelopes | Memória, controle e dependências interpretáveis pelo núcleo |
| Proveniência | Correlação de origens e de fatos derivados |
| Conformidade | Casos positivos, negativos, incompletos e adversariais |
| Redução | Tradução equivalente para capacidades conhecidas, quando existir |
| Migração | Efeitos sobre publicações e consumidores anteriores |

“Payload livre” sem semântica não é extensão precisa. Atributos podem ser armazenados de qualquer forma, mas seus significados e tipos devem ser definidos. O consumidor não precisa conhecer uma classe concreta para interpretar o envelope.

## 3. Três formas de consumo

Um consumidor pode interpretar precisamente a extensão, consumir uma redução já publicada para operações conhecidas ou aplicar seu envelope conservador. Deve declarar qual forma utilizou. Se nenhuma forma for possível, deve emitir incompatibilidade explícita e conservar os fatos independentes cujo escopo permita consumo.

Redução e operação original NÃO DEVEM ser executadas como dois eventos. A representação executável selecionada deve ser única; a original pode permanecer como evidência não executável. Reuniões de efeitos não podem contar a mesma ocorrência duas vezes.

Consumidor desconhecedor da extensão não pode ignorá-la, considerar seus resultados nulos ou inferir ausência de efeitos. Falta de envelope é incompatibilidade, não autorização para tratá-la como `nop`.

Um domínio de extensão identificado é `known(opaque_type(id, version))`, ou a forma de tipo definida por uma extensão padronizada, envolvida em `known`. O manifesto determina operações disponíveis; o núcleo não presume igualdade, aritmética, codificação ou conversões. Leitura e cópia de valor entre células do mesmo domínio conhecido não exigem interpretar sua estrutura interna. Consumir conservadoramente não muda a identidade desse domínio para `unknown_type`, e uma extensão não pode redefinir tipo desconhecido como domínio opaco.

## 4. Compatibilidade possui dimensões

| Dimensão | Critério |
| --- | --- |
| Estrutural | A publicação continua decodificável/validável no contrato abstrato |
| Semântica | Operações existentes preservam significado |
| Conservadora | Um consumidor anterior pode manter análise segura por fallback |
| De precisão | O consumidor anterior mantém os mesmos limites e resultados exigidos |
| De transporte | A representação física pode ser lida pela versão compatível do adapter |

Adicionar operação com envelope pode preservar compatibilidade conservadora e perder precisão em consumidor antigo. Isso NÃO equivale a garantir “nenhuma quebra”. Uma mudança que requer compreensão da nova operação deve declarar a capacidade como requerida para aquele perfil/consulta.

## 5. Versionamento

A versão semântica usa `major.minor.patch`.

Uma correção editorial sem mudar obrigação ou comportamento pode aumentar `patch`. Uma adição de metadado não semântico ou capacidade opcional explicitamente negociada pode aumentar `minor`, preservando o significado anterior. Mudança em identidade, domínio de tipo, ordenação, alias, avaliação, controle, completude ou regra de validade que invalide publicações/consumidores anteriormente conformes exige `major`, salvo se isolada em nova versão de extensão sem substituir a anterior.

Mudar o default de desconhecido para vazio é mudança semântica incompatível. Alterar um código humano de diagnóstico não é, desde que código tipado e significado permaneçam. Uma errata semântica não deve ser ocultada como correção apenas editorial.

### 5.1 Correção de conhecimento de tipo — 2.0.0

A 1.0.0 exigia manter objetos de tipo desconhecido, mas só catalogava domínios identificados; `unknown(T,...)` ainda exigia `T`, e I-08 condicionava a verificação de tipos a estarem conhecidos. `TYPE_UNKNOWN` como razão, sozinho, não definia a representação desses objetos. `opaque_type` já designava domínio de extensão e não preenchia essa lacuna.

A 2.0.0 introduz `TypeRef` como conhecimento obrigatório de domínio, generaliza os pontos pertinentes e torna explícita a rejeição de operações precisas cujas precondições de tipo não sejam satisfeitas. Não é correção editorial `1.0.1`: muda o contrato abstrato e as obrigações de validadores e consumidores do núcleo fechado. Também não é capacidade opcional isolada que justifique apenas `minor`. Pela regra acima, exige `major`, mesmo preservando o significado das operações sobre domínios já conhecidos.

A mesma edição distingue identificação concreta de domínio de igualdade relacional `sameDomain`. Cópias e transmissões podem conservar sua relação de valor com domínio não identificado, mediante prova declarativa ou regras de identidade/alias; operadores que interpretam o domínio mantêm suas exigências concretas. Premissas tipadas e obrigações de conformidade por sub-requisito fazem parte desse contrato 2.0.0, sem transformar lacunas em variáveis de unificação.

Na migração, um domínio `T` estabelecido passa a `known(T)`, inclusive tipos de extensão. Um fato realmente sem domínio concreto estabelecido passa a `unknown_type(u)` com lacuna explícita; não se renomeia um domínio de extensão para representar essa ausência. Cópias conhecidas usam prova de `sameDomain` quando disponível; operações que interpretem o domínio exigem o tipo concreto. Construções sem suas precondições precisam de abstração com envelopes e operandos preservados. Publicações devem ser revalidadas; um consumidor 1.0.0 não recebe garantia automática de fallback para as novas formas.

Os perfis passam a `@2` para refletir as obrigações de `TypeRef` e os novos oráculos, inclusive perfis que herdam o modelo estrutural. As extensões padronizadas mantêm `memory.regions@1`, `control.local@1` e `control.indirect@1`: suas regras próprias de bytes/codecs, frames e labels não mudam; nesta edição são usadas sob o contrato de núcleo 2.0.0. Sua versão isolada não autoriza compatibilidade entre versões major da publicação.

## 6. Negociação

Antes de alegar conformidade precisa, o consumidor verifica versão major, perfis e versões de capacidades usadas. Para cada capacidade, registra `precise`, `reduced`, `conservative` ou `unsupported`. Essa verificação não depende de descobrir conteúdo desconhecido por tentativa de execução.

Um estado de enumeração desconhecido não recebe interpretação automática. Quando a compatibilidade conservadora foi definida, usa-se seu envelope; caso contrário, o escopo é indisponível. Publicações de major diferente não podem ser interpretadas como iguais apenas porque a estrutura física parece semelhante.

## 7. Evolução de fatos e resultados

Uma nova versão do produtor pode substituir abstração por operações mais precisas, acrescentar declaração antes indisponível ou restringir um alvo aberto. Isso pode remover candidatos espúrios em resultados derivados. Portanto, resultados de dependências não são necessariamente conjuntos que apenas crescem entre revisões.

A propriedade exigida é preservação do comportamento concreto e das evidências independentes válidas, não monotonicidade textual de todos os resultados. Uma evidência revogada ou um valor morto por análise mais precisa deve ser identificado como tal, não mantido como fato corrente para evitar diferenças.

## 8. Neutralidade de linguagem

Uma extensão não deve impor à IR principal a hierarquia sintática de uma linguagem. Conceitos como memória sobreposta, transferência indireta e conclusão de trecho podem ter semântica independente. Um comportamento que permanece genuinamente específico pode ser extensão qualificada com redução/envelope; não precisa contaminar o núcleo.

A aceitação de um segundo produtor requer equivalência das observações normalizadas, não igualdade de grafia, ordem interna de IDs ou quantidade de operações quando a decomposição legítima difere. Proveniência continua distinguindo origens.

## 9. Transformações posteriores

Normalização, especialização ou eventual forma SSA podem existir em produtos/revisões separados. A V2 não exige SSA nem proíbe sua construção posterior. Qualquer transformação deve conservar memória observável, ordem de interação, controle, restantes e proveniência, além de definir como pontos antigos se correlacionam com novos.

Uma otimização não pode eliminar uma operação apenas por o consumidor não conhecer sua extensão. Ausência de efeitos e possibilidade de especulação não são inferidas do nome ou de resultados não utilizados.
