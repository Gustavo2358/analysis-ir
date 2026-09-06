# Analysis IR — Especificação V2

**Versão:** 2.0.0

**Natureza:** especificação de representação intermediária para análise estática  
**Idioma:** português; identificadores semânticos em inglês  
**Escopo:** programas imperativos sequenciais, memória mutável, controle explícito e observação de dependências

Esta edição corrige a lacuna de tipos desconhecidos da 1.0.0: `TypeRef` distingue `known(T)` de `unknown_type(u)`, preservando entidades e leituras sem liberar operações que exigem domínio conhecido. `opaque_type` continua identificando domínio de extensão. A mudança exige versão major e perfis `@2`, conforme a [decisão de compatibilidade e migração](especificacao/09-extensibilidade-e-compatibilidade.md#51-correção-de-conhecimento-de-tipo--200).

## Propósito

Analysis IR estabelece um contrato independente de linguagem entre produtores de conhecimento semântico e consumidores de análise. O contrato permite construir grafos de fluxo de controle, interpretar efeitos de memória, calcular definições alcançáveis e valores possíveis e identificar dependências com evidência e incompletude explícitas.

**O critério central é a suficiência bilateral:** cada capacidade deve ser sustentada por fatos que um produtor possa estabelecer e por observações que um consumidor realmente necessite. Uma capacidade não é acrescentada apenas por analogia com uma linguagem; uma informação necessária à análise não é omitida apenas porque um produtor ainda não a fornece.

A V2 fixa um núcleo de operações sobre estado mutável e transferências explícitas. Não exige forma SSA, implementação de compilador, formato de transporte ou biblioteca de grafos. As sequências da IR são unidades de representação, não a identidade dos blocos básicos de um produto CFG. Resultados de análise permanecem produtos derivados.

## Organização e ordem de leitura

| Documento | Conteúdo | Natureza |
| --- | --- | --- |
| [00 — Escopo e convenções](especificacao/00-escopo-e-convencoes.md) | Significado normativo, garantias e limites | Normativa |
| [01 — Modelo e identidades](especificacao/01-modelo-e-identidades.md) | Publicação, unidades, entradas, declarações e pontos | Normativa |
| [02 — Tipos, valores e operandos](especificacao/02-tipos-valores-e-operandos.md) | Valores, expressões, referências e avaliação | Normativa |
| [03 — Memória e aliases](especificacao/03-memoria-e-aliases.md) | Células, regiões, vistas, sobreposição e inicialização | Normativa |
| [04 — Operações](especificacao/04-operacoes.md) | Catálogo completo do núcleo e contratos externos | Normativa |
| [05 — Controle e invocações](especificacao/05-controle-e-invocacoes.md) | Transferências, exceções, retornos e extensões locais | Normativa |
| [06 — Incompletude e proveniência](especificacao/06-incompletude-e-proveniencia.md) | Cobertura, envelopes conservadores e rastreabilidade | Normativa |
| [07 — Contrato de produtores](especificacao/07-contrato-de-produtores.md) | Obrigações de lowering e independência do produtor | Normativa |
| [08 — Contrato de consumidores](especificacao/08-contrato-de-consumidores.md) | CFG, efeitos, reaching definitions, valores e dependências | Normativa |
| [09 — Extensibilidade](especificacao/09-extensibilidade-e-compatibilidade.md) | Extensões tipadas, negociação e evolução | Normativa |
| [10 — Perfis V2](especificacao/10-perfis-de-conformidade.md) | Capacidades obrigatórias, opcionais e níveis de aceitação | Normativa |
| [11 — Rastreabilidade bilateral](especificacao/11-rastreabilidade-bilateral.md) | Requisitos de entrada, observação e oráculos | Normativa |
| [Notação dos exemplos](exemplos/00-notacao.md) | Sintaxe de leitura e convenções compartilhadas | Informativa |
| [Fluxo e valores](exemplos/01-fluxo-e-valores.md) | Atribuição, bifurcação, ciclos, seleção e cópia de valor | Informativa |
| [Memória e chamadas](exemplos/02-memoria-e-chamadas.md) | Aliases, bytes, escritas parciais, chamadas e recursos | Informativa |
| [Extensões e parcialidade](exemplos/03-extensoes-e-parcialidade.md) | Controle local, indireto, desconhecido e evolução | Informativa |
| [Conhecimento de tipo](exemplos/04-conhecimento-de-tipo.md) | Domínios conhecidos, extensão, tipo/valor desconhecidos e precondições | Informativa |
| [Invariantes](conformidade/01-invariantes.md) | Regras de validade e falhas detectáveis | Normativa |
| [Oráculos](conformidade/02-oraculos.md) | Cenários de conformidade com resultados esperados | Normativa |
| [Referências](REFERENCIAS.md) | Fundamentos e fontes conceituais | Informativa |

## O que significa “V2 completa”

O conjunto define integralmente o vocabulário, a semântica, as regras de validade, os envelopes de desconhecimento, os perfis e os critérios de conformidade da versão 2.0.0. Não significa cobertura integral de qualquer linguagem ou precisão perfeita para todo programa. Uma publicação pode ser válida e explicitamente parcial. Uma implementação pode declarar apenas os perfis que satisfaz.

O núcleo inclui representações conservadoras para fatos indisponíveis. As extensões padronizadas `memory.regions@1`, `control.local@1` e `control.indirect@1` têm semântica definida nesta versão, mas sua implementação não é exigida do perfil escalar. Um consumidor que não as interprete deve usar o fallback normativo ou emitir incompatibilidade explícita.

## Decisões de representação

A V2 adota identidades próprias; declarações nominais separadas de armazenamento; expressões puras avaliadas no ponto de uso; atribuições a estado mutável; sequências terminadas por transferências; contratos de chamada e efeitos desconhecidos explícitos; e resultados de análise externos à IR. Essas são decisões semânticas, não escolhas de classes, algoritmos ou persistência.

A identificação de uma dependência literal não precisa aguardar análise de valores. A identificação de uma dependência calculada exige o valor do operando no ponto correto, a interpretação do recurso e as condições de completude aplicáveis. Nenhum resultado finito pode ocultar um restante desconhecido.
