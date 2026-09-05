# Referências e bases conceituais

**Informativo.** As fontes abaixo fundamentam conceitos de especificação, análise estática e representações intermediárias. Elas não aprovam esta especificação nem impõem sua implementação. O significado normativo da Analysis IR é definido pelos documentos deste conjunto; não há incorporação automática de semânticas externas.

## Convenções de requisitos

**Bradner, S.** *Key words for use in RFCs to Indicate Requirement Levels*. RFC 2119, BCP 14, 1997. [RFC Editor](https://www.rfc-editor.org/info/rfc2119/).  
**Leiba, B.** *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. RFC 8174, BCP 14, 2017. [RFC Editor](https://www.rfc-editor.org/info/rfc8174/).

A distinção entre obrigação, recomendação e permissão inspira o vocabulário DEVE/DEVERIA/PODE. As definições locais em português estão em [00](especificacao/00-escopo-e-convencoes.md).

## Dataflow e interpretação abstrata

**Kildall, G. A.** *A unified approach to global program optimization*. POPL, 1973, pp. 194–206. DOI [10.1145/512927.512945](https://doi.org/10.1145/512927.512945).

O trabalho apresenta um enquadramento geral de análise global por fluxo de programa. É base conceitual para distinguir representação do programa, estrutura de fluxo e propriedades calculadas; não exige que o contrato adote um algoritmo específico.

**Cousot, P.; Cousot, R.** *Abstract interpretation: a unified lattice model for static analysis of programs by construction or approximation of fixpoints*. POPL, 1977, pp. 238–252. [Página dos autores](https://www.di.ens.fr/~cousot/COUSOTpapers/POPL77.shtml).

A distinção entre comportamento concreto e abstração conservadora fundamenta a inclusão de comportamentos, o tratamento de aproximações e a separação entre conjunto enumerado e limite de conhecimento. O domínio exato de análise não é prescrito pela IR.

**Reps, T.; Horwitz, S.; Sagiv, M.** *Precise interprocedural dataflow analysis via graph reachability*. POPL, 1995, pp. 49–61. DOI [10.1145/199448.199462](https://doi.org/10.1145/199448.199462). [Resumo dos autores](https://research.cs.wisc.edu/wpis/abstracts/popl95.abs.html).

O trabalho trata uma classe de problemas interprocedurais com domínio finito de fatos e funções distributivas. É referência para a importância de caminhos e relações interprocedurais. Esta especificação não afirma que todo o seu domínio de valores ou storage satisfaz as restrições daquela classe.

## Representação e extensão semântica

**LLVM Project / MLIR.** *MLIR Language Reference*. [Documentação oficial](https://mlir.llvm.org/docs/LangRef/).

A documentação exemplifica separação entre operações, tipos, blocos, regiões e propriedades de validade. Analysis IR adota seu próprio modelo de estado mutável e não importa exigência de SSA ou hierarquia de regiões de MLIR.

**LLVM Project / MLIR.** *Side Effects & Speculation*. [Documentação oficial](https://mlir.llvm.org/docs/Rationale/SideEffectsAndSpeculation/).

A modelagem explícita de efeitos motiva distinguir desconhecimento de ausência de efeito e não autorizar transformações por resultados não utilizados. A semântica de envelopes, regiões e chamadas neste conjunto é própria.

**LLVM Project / MLIR.** *Dialect Conversion*. [Documentação oficial](https://mlir.llvm.org/docs/DialectConversion/).

Conversão entre representações e requisitos de legalidade são referências conceituais para negociação e redução de capacidades. Nenhuma API, infraestrutura de conversão ou formato físico é exigido pela Analysis IR.

## Autoridade e aplicação

Uma especificação de linguagem ou ambiente de origem continua sendo autoridade para a tradução daquele produtor. Seus conceitos devem ser normalizados para os contratos da IR ou preservados como abstração explícita. Um consumidor genérico não deve depender de consultar essa especificação para completar semântica ausente na publicação.

As decisões particulares de Analysis IR — núcleo não SSA, pontos antes/depois, perfis de precisão, envelopes e extensões padronizadas — são definidas aqui como contrato independente. Semelhança conceitual com outra representação não implica compatibilidade direta.
