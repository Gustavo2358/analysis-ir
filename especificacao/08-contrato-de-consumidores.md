# 08 — Contrato de CFG, efeitos, dataflow e dependências

**Analysis IR 2.0.0 — Normativo**

## 1. Regras comuns

Um consumidor DEVE declarar versão, perfis, extensões interpretadas, escopo da consulta, premissas e publicação utilizada. Não deve depender do produtor original, de texto-fonte ou de detalhes de transporte para derivar semântica.

Cada resultado é um produto separado. Seus IDs correlacionam-se com pontos, operações, operandos, objetos e origens IR; não substituem os IDs da publicação. Atualizar CFG, storage ou premissas invalida resultados dependentes que não foram demonstrados compatíveis.

## 2. Construção de CFG

O CFG DEVE derivar successors das operações e terminadores. A execução sequencial vale apenas dentro de cada sequência. Entradas e saídas normais/excepcionais devem permanecer distinguíveis. Chamadas com múltiplos resultados e operações opacas podem exigir separar pontos anteriores e posteriores.

O particionamento em blocos básicos é livre, desde que preserve pontos observáveis, a relação de transição e o mapeamento para operações. A ausência de bloco para uma operação executável representada precisa ser explicada por uma transformação semanticamente válida, não por falta de suporte.

Predicado desconhecido admite os dois destinos; controle aberto exige fronteira conservadora. Return e halt não recebem successor de fallthrough. Invocação local/interunidade exige pareamento de retorno ou marcação da sobreaproximação adotada.

**Consulta conceitual:** `successors(point, context)` deve identificar destino, condição/tag quando conhecida, vínculo de invocação/retorno e restante aberto. Não é uma API física prescrita.

## 3. Statement Effects e Storage Semantics

O consumidor de storage DEVE resolver os locais normalizados em células/intervalos/alternativas e restante aberto, sem assumir independência nominal. O consumidor de efeitos DEVE distinguir leitura de valor, leitura de endereço, escrita possível, escrita obrigatória, intervalo e completude.

Efeitos derivam da semântica das operações e de limites externos. `assign` lê origem/endereço e escreve destino; `branch` lê o predicado; `invoke` lê target e argumentos por valor antes dos efeitos da interação; referências passadas disponibilizam locais; resultados normais são escritos apenas nessa transição.

Para operações opacas, o consumidor aplica os envelopes. Não pode tratar falta de interpretação como efeito vazio. Escritas obrigatórias e possíveis não são intercambiáveis. O produto de efeitos registra a justificativa de strong/weak update por local.

## 4. Reaching definitions

Uma definição é um evento que atribui ou pode atribuir valor a uma localização. Sua identidade derivada deve distinguir operação, destino/intervalo e resultado de controle. Eventos de entrada e efeitos desconhecidos também podem produzir definições, sem criar operações-fonte fictícias.

A consulta conceitual é:

```text
reachingDefinitions(place, before|after(operation,outcome), entryScope, context)
  → definitions + contributedRanges + unknownRemainder + reachability + evidence
```

`reachability` distingue ponto alcançável, comprovadamente inalcançável e alcançabilidade não determinada. Vazio em ponto inalcançável não equivale a valor desconhecido em ponto alcançável. Um local possivelmente não inicializado conserva definição de entrada apropriada.

### 4.1 Obrigação extensional

Se um comportamento admitido chega ao ponto consultado e um evento pode fornecer bytes/valor ainda não sobrescritos do local, o resultado DEVE incluir esse evento ou um restante desconhecido que cubra explicitamente sua contribuição. Um evento completamente sobrescrito em todos os caminhos admitidos não pode ser apresentado como definição corrente precisa.

### 4.2 Caso escalar fechado

Para CFG fechado e efeitos exatos sobre células independentes, as relações clássicas são:

```text
IN(n)  = união dos OUT(p,resultado) dos predecessores válidos de n
OUT(n) = GEN(n) ∪ (IN(n) − KILL(n))
```

O estado das entradas é adicionado nos pontos de entrada pertinentes. Nós não alcançados não recebem seeds automaticamente. `GEN` e `KILL` são resultados derivados, não campos da IR.

Uma atribuição obrigatória a célula exata elimina suas definições anteriores. Uma escrita possível ou destino alternativo conserva definições que podem sobreviver. Para efeitos por resultado, as equações se aplicam às transições correspondentes, não a uma união indiferenciada antes de sair da operação.

### 4.3 Regiões e escritas parciais

A unidade de eliminação é o intervalo comprovadamente sobrescrito. Uma leitura maior pode reunir fragmentos provenientes de várias definições. O resultado não pode marcar a definição antiga como fornecedora de todos os bytes quando parte foi sobrescrita, nem apagá-la de bytes não modificados.

Um consumidor pode particionar memória ou usar representação simbólica. Precisa conservar o significado por intervalos, não implementar um algoritmo específico.

### 4.4 Ciclos e chamadas

Ciclos exigem solução conservadora estável ou restante explícito por limite. Uma única passagem em ordem textual não satisfaz o contrato geral. Contextos de retorno devem ser respeitados no perfil preciso aplicável.

O conjunto de definições é separado do conjunto de valores. Encontrar a definição `assign(y,read(x))` exige consultar o valor de `x` **antes dessa atribuição**, e não o valor de `x` no ponto de uso posterior de `y`.

## 5. Valores possíveis

A consulta conceitual é:

```text
possibleValues(expressionOrPlace, point, entryScope, context)
  → enumeratedValues + unknownRemainder + reachability + precision + evidence
```

Os valores enumerados são candidatos justificados pelas regras abstratas e pelas definições pertinentes. Não são, automaticamente, testemunhos de caminhos concretamente viáveis. Um consumidor deve declarar a precisão de caminho/contexto empregada.

### 5.1 Significado do resultado

Para `TypeRef=known(T)`, resultado finito fechado `V` representa valores contidos em `V`. Resultado `(V, unknownRemainder=true)` admite outros valores de `T` e conserva `V` como evidência enumerada. Do ponto de vista puramente denotacional, o restante aberto pode representar todo `T`; conservar `V` acrescenta rastreabilidade, não restringe indevidamente esse domínio.

Para `unknown_type(u)`, o consumidor conserva a lacuna de domínio e não interpreta o restante como pertencente a `int`, `text`, `bytes` ou a uma extensão escolhida por conveniência. Candidatos sustentados, por exemplo por alternativas de `choice` de tipos conhecidos distintos, conservam seus próprios domínios e evidência. Um resultado fechado exige justificar exaustividade também dessas alternativas; ausência de candidatos com domínio desconhecido em ponto alcançável não significa conjunto vazio fechado. Refinamento derivado de conhecimento não altera retroativamente o `TypeRef` da publicação nem valida operação que violava uma precondição.

Conhecer o domínio de `opaque_type` sem interpretar a extensão mantém a identidade desse domínio e o modo de consumo negociado. Não deve ser relatado como domínio não identificado. Em todos os casos, leituras, ocorrências, aliases e origens conhecidos permanecem disponíveis, independentemente da precisão dos valores.

Conjunto vazio fechado em ponto alcançável só é válido se o domínio consultado realmente não admite valor; normalmente indica inconsistência. Ponto inalcançável possui estado próprio. Falta de inicialização e entrada externa geram valor desconhecido, não conjunto vazio fechado.

### 5.2 Transferência e reunião

Literais e operações conhecidas são interpretados segundo o catálogo. Reuniões unem candidatos e propagam o restante aberto. Combinações cartesianas sem correlação podem introduzir candidatos espúrios e devem permanecer classificadas como possibilidades abstratas.

Uma escrita forte substitui o valor corrente; uma escrita fraca reúne alternativas. Desconhecimento do resultado de uma escrita obrigatória não conserva automaticamente o literal anterior. Um efeito apenas possível pode preservá-lo através do caminho sem escrita.

Capturas são respeitadas: uma cópia de valor reflete o estado na definição, não uma referência tardia à expressão original. Decodificação, preenchimento e truncamento são aplicados antes de interpretar um nome externo. Não se remove padding por conveniência.

### 5.3 Terminação

Domínios de valores podem crescer indefinidamente. A especificação exige terminação operacional declarada ou resultado parcial explícito, não enumeração infinita. Aproximações e limites não podem perder valores possíveis sem restante. Os oráculos finitos dos perfis devem ser satisfeitos com a precisão exigida por eles.

## 6. Fatos de dependência

Uma dependência deve identificar artefato/unidade de origem, site de interação ou relação estrutural, categoria e ação, namespace do alvo, identidade interna ou nome observado/calculado, evidência, modo de observação e completude.

São modos distintos:

| Modo | Alegação |
| --- | --- |
| `STRUCTURAL` | Relação de artefato estabelecida sem execução |
| `OBSERVED` | Uma interação está representada no inventário, independentemente de alcançabilidade |
| `MAY_EXECUTE` | A interação pode ser alcançada no modelo declarado |
| `ENUMERATED_TARGET` | Um nome/identidade é candidato derivado para o site |

Os modos podem coexistir; não são uma escala de certeza única. “Possível” não deve ser convertido em “executado”. Uma interaction em código inalcançável ainda pode ser observada no inventário, mas não recebe automaticamente `MAY_EXECUTE`.

### 6.1 Alvos diretos e calculados

Um nome literal observado pode gerar fato nominal sem reaching definitions. Um alvo calculado exige o valor de seu operando em `before(invoke)`. A leitura desse operando ocorre antes de qualquer escrita causada pela própria interação.

Resolver um nome contra um catálogo é passo distinto. Um nome de arquivo lógico não determina automaticamente caminho físico. Um schema usado estruturalmente não prova leitura de todas as tabelas que descreve. Uma chamada a um serviço não prova alvo indireto de protocolo sem regra de interpretação desse protocolo.

### 6.2 Extração especializada

Consumidores especializados podem consultar valores de argumentos ou fatias de regiões e aplicar contratos de recursos/protocolos. DEVEM reutilizar os produtos canônicos de CFG, storage, efeitos e valores. Não podem reconstruir fluxo por texto nem escolher a última atribuição aparente.

O resultado conserva targets enumerados e restante desconhecido. A IR não contém regras proprietárias de nomeação ou campos de protocolos específicos; contém os operandos, regiões e sites suficientes para essas interpretações.

### 6.3 Completude

“Todas as dependências” só pode significar todas as dependências representáveis no escopo, nas entradas e no modelo explicitados, incluindo a indicação de classes ainda abertas. Uma alegação finita exaustiva exige inventário completo, controle relevante fechado, efeitos/storage suficientes, avaliação do target sem restante e interpretação/catálogo suficientes para a espécie de fato alegada.

Lacuna global não apaga observações independentes. Tampouco uma observação independente autoriza completar o restante global.

Uma consulta DEVE declarar se enumera dependências diretas dos sites e artefatos representados ou também um fechamento transitivo entre unidades/recursos. Observar chamada a uma unidade sem corpo permite a relação direta, não enumeração completa das dependências internas daquela unidade. Completude transitiva exige cobertura dos corpos, contratos e catálogos transitivamente relevantes.

## 7. Análises adicionais

O CFG e a proveniência podem alimentar dominância, pós-dominância, dependência de controle ou slicing como produtos separados. Saídas anormais, divergência, múltiplas entradas e fronteiras abertas devem entrar no escopo dessas análises. A IR V2 não publica esses resultados nem pressupõe um algoritmo único.

## 8. Escala e execução sob demanda

Consultas por site, operando, objeto ou intervalo são permitidas. Resultados sob demanda e análise completa devem ser semanticamente compatíveis sob as mesmas premissas e precisão. Não se exige materializar um grafo global de todos os artefatos para obter um fato nominal local.

A representação não impõe complexidade quadrática, cópia de todas as definições por ponto ou expansão física de tabelas. Limites de recursos só podem afetar precisão/completude de forma observável; nunca a cardinalidade publicada de fatos já cobertos.
