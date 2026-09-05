# 10 — Perfis e níveis de conformidade V1

**Analysis IR 1.0.0 — Normativo**

## 1. Papéis

`Producer` emite publicações conformes. `Validator` verifica regras decidíveis de integridade e registra limites de validação semântica. `Consumer` produz um resultado declarado: estrutura/CFG, storage/efeitos, reaching definitions, valores ou dependências.

A declaração de conformidade especifica papel, versão, perfis e capacidades, não apenas “V1”. Validar referências e tipos não certifica por si só que o lowering preserva toda a semântica da entrada.

## 2. Níveis de aceitação

| Nível | Exigência |
| --- | --- |
| `VALID` | Publicação satisfaz invariantes do modelo e declara capacidades/lacunas |
| `CONSERVATIVE` | Consumidor preserva possibilidades por semântica conhecida ou envelopes |
| `PRECISE_FOR_PROFILE` | Além de conservador, satisfaz todos os oráculos de precisão do perfil |

Um consumidor pode ser preciso para um subconjunto da publicação e conservador para outro, desde que o escopo e a combinação nas consultas permaneçam explícitos. O restante aberto não impede `VALID`; pode impedir `PRECISE_FOR_PROFILE` para uma consulta que o atravessa.

## 3. Perfil `AIR-STRUCTURE@1`

Exige modelo fechado, identidades, múltiplas unidades/entradas, sequências, origens, cobertura, transferências diretas, `invoke`, `return`, `raise`, `halt` e interpretação estrutural de `opaque`.

O consumidor CFG desse perfil deve preservar ordem intrassequência, bifurcações, dispatch, ciclos, resultados de interação, saídas e fronteiras abertas. Não precisa determinar valores de predicados nem layout físico. Deve preservar os operandos relevantes para consumidores posteriores.

Oráculos obrigatórios: O-01 a O-10, O-18 a O-22, O-29 a O-34 e O-41 a O-48, no escopo estrutural de cada um.

## 4. Perfil `AIR-SCALAR-FLOW@1`

Inclui `AIR-STRUCTURE@1` e acrescenta células abstratas com associação/independência explícita, objetos com alias de célula, estados de entrada, `assign`, `havoc`, `nop` e todas as expressões de núcleo com seus tipos e precondições.

O consumidor de efeitos deve distinguir reads/writes e strong/weak update. O consumidor RD deve satisfazer os cenários escalares fechados, inclusive ciclos finitos de definições e cópia de valor. O consumidor de valores deve satisfazer os oráculos finitos de literais, expressões, reuniões e restante desconhecido. Não há obrigação de precisão relacional geral ou enumeração de domínios infinitos.

Calls podem ser tratados por limites externos, sem corpo interprocedural preciso. Precisão pós-chamada depende do limite de efeito adequado. O target na entrada é observável mesmo quando efeitos posteriores são desconhecidos.

Oráculos adicionais: O-11 a O-17, O-23 a O-28, O-35, O-36, O-49 e O-50, além dos cenários escalares de O-01 a O-10.

## 5. Perfil `AIR-REGION-FLOW@1`

Inclui `AIR-SCALAR-FLOW@1` e `memory.regions@1`. Exige vistas, codecs padronizados, intervalos, sobreposição, escrita parcial, `copy_bytes`, associação aberta e acessos calculados com limites explícitos.

O consumidor RD deve identificar contribuições por intervalo. O consumidor de valores deve interpretar os cenários finitos de bytes e codecs conhecidos. Codec desconhecido deve preservar cópia bruta e bloquear apenas interpretação não sustentada. Não é obrigatório deduzir layout ausente de uma linguagem-fonte.

Oráculos adicionais: O-37 a O-40, O-51 a O-55.

## 6. Perfil `AIR-LOCAL-CONTROL@1`

Inclui `AIR-STRUCTURE@1` e `control.local@1`. Exige interpretação de frames locais, portas de conclusão, default, resume e unwind; deve preservar compartilhamento de ativação e retornos correspondentes.

Um consumidor RD que declare precisão nesse perfil também necessita `AIR-SCALAR-FLOW@1` ou `AIR-REGION-FLOW@1`, conforme os locais usados. Consumir o fallback com retornos indiscriminados é conformidade conservadora, não precisão desse perfil.

Oráculos adicionais: O-56 a O-60.

## 7. Perfil `AIR-INDIRECT-CONTROL@1`

Inclui `AIR-STRUCTURE@1` e `control.indirect@1`. Exige tipos de label com limite fechado e CFG conservador inicial sobre todo o limite. Redução dos targets por dataflow é opcional; quando realizada, exige revisão consistente e ausência de subaproximação.

Oráculos adicionais: O-61 a O-63.

## 8. Perfil `AIR-DEPENDENCY-OBSERVATION@1`

Inclui `AIR-STRUCTURE@1` para sites executáveis e modelo de relações estruturais para artefatos. Exige preservar espécie de interação, target, namespace, ponto, origem e restante. Fatos literais/estruturais não exigem perfil de dataflow.

Para alegar targets calculados precisos, o consumidor deve declarar o perfil de fluxo pertinente e seus resultados utilizados. Para dependência de catálogo, deve declarar cobertura desse catálogo. Não há interpretação precisa automática de protocolos especializados.

Oráculos adicionais: O-23, O-24, O-35, O-36, O-45, O-46, O-64 a O-68.

## 9. Capacidades independentes de cardinalidade

Perfis delimitam classes de comportamento. Não impõem quantidade máxima de atribuições, chamadas, declarações, branches, entradas ou aliases. Um produtor não pode afirmar um perfil para uma unidade e publicar somente sua primeira ocorrência suportada.

Uma implementação inicial pode declarar menos perfis. Isso não modifica o significado da V1 nem permite representar construções fora do perfil como se fossem suportadas. A negociação e o fallback se aplicam a cada publicação e escopo de consulta.

## 10. Evidência de conformidade

A evidência deve conter identificação do perfil, cenários, resultados esperados e observados, falhas, limites, publicação e premissas. Os oráculos são especificações de resultados, não uma alegação de implementação existente.

A escolha de framework de testes, formato de casos, mecanismo de execução e armazenamento de evidências é externa a esta especificação. O resultado deve ser reprodutível semanticamente.
