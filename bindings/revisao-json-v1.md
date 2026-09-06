# Revisão normativa do JSON Binding v1 e handoff

**Informativo — PR #2, `feat/adiciona-contrato-json`.** A autoridade é a AIR;
este registro explica decisões, sem substituir os documentos normativos.
O binding permanece **DRAFT**, destinado a AIR 2.0.0; não é aceito implicitamente
pelo merge. Implementação, transporte e versão semântica são contratos distintos.

## Evidência e limite da revisão

Foram lidos README, todos os documentos de `especificacao/`, invariantes, oráculos,
exemplos e a candidata JSON integral. Baseline da AIR/`main`:
`0b2fbce7046010b22b32efa8cbc3e75ccba09442`; candidata original do PR:
`4148e28a778ac64eba3c51227288e9738fade445`.

A implementação `air-java` foi consultada somente como evidência, no snapshot local
`2108294d9dfeb89d0019ce75fab27172b15a75b9`: README, status de implementação,
`Interactions`, `Proofs`, `Operations`, `Places`, `Memory`, `Expressions` e checks
de referências/operações pertinentes. Ela reconhece `contracts` e `SafetyAssertion`
como decisões da representação Java; isso não fornece autoridade normativa.
Nenhum arquivo de `air-java` faz parte deste trabalho.

## Decisões e ação futura

| Divergência | Decisão normativa | Motivo | Ação futura no air-java |
| --- | --- | --- | --- |
| Contracts: `Publication.contracts` / `ContractId` / `Contract` | Não integrar o inventário ao núcleo. `ContractRef` é valor de autoridade, versão e evidência; assinatura externa fica no `invoke`, efeitos/outcomes e premissas já estão materializados. | AIR 01, §9 já proíbe referência sem conteúdo e lookup. Produtor fornece fatos; consumidor observa esses fatos sem precisar de catálogo de contratos. Internamento compartilhado é conveniência, não entidade bilateral necessária. | Remover inventário/IDs do contrato público AIR; colocar assinatura externa no site, converter referência para valor rastreável e atualizar índices/traversal/validação. Não descartar fatos dos contratos antigos: materializá-los em cada uso e revalidar. |
| `ResourceTarget(ResourceId)` | Remover a quarta variante executável. Manter declarações `ResourceId` e targets interno, literal e calculado. | AIR 01/04 já preserva o significado com essas três formas. Produtor pode repetir uma descrição; consumidor precisa do uso/site, não do atalho. Recurso declarado não resolve catálogo nem identifica entrada. | Expandir descrições nos usos, preservando origens e ocorrências; target interno só por `EntryId`. Manter inventário de recursos sem usar seu ID como execução. |
| `DisjointStorage` | Formalizar somente `disjoint_storage(StorageId[])`, par a par e universal na publicação, sem campo de escopo seletivo. | Independência já é normativa em AIR 03, B-08/B-14, I-12, O-14/O-15. O produtor estabelece separação; o consumidor precisa excluir influência de escrita limitada. Associações não afirmam separação entre bases distintas; intervalos da mesma base já têm representação suficiente. | Reconciliar com 03, §3.1: remover `FactScope` genérico, verificar bases/unicidade/evidência e não certificar verdade física. Garantia restrita a caminho não migra para universal. |
| `SafetyAssertion` / `SafetyProperty` | Remover do núcleo/binding. Manter obrigações de pureza, totalidade, codec e domínio em suas regras; certificados pertencem à evidência de validação. | A AIR exige precondições, mas não esses cinco tokens. Não há necessidade bilateral de enums de testes. Garantia de escolha já usa `sameDomain`; igualdade de extensão exige manifesto. | Remover dependência de tokens para aceitar AIR válida; preservar diagnósticos de contradição e `INCOMPLETE_VALIDATION` para obrigações não verificadas. Eventual linguagem de provas exige extensão própria. |
| `boundsProof`, `accessProof`, `knownRemainderDomainProof` | Remover campos semânticos de expressões, Places e ByteRange. | Referência privada a certificado não define nem substitui precondição; `sameDomain` tem sujeitos/sites consultáveis. | Atualizar construtores e checks para usar fatos normativos aplicáveis, sem exigir ponteiro privado nem aprovar só por presença de ID. |
| `ContractName` e contrato de `ExtensionCodec` | Regra precisa exige capacidade/manifesto; `ContractRef` de interação não é regra de nome ou codec. | Autoridade/versão sozinha não diz como transformar nome, codificar bytes ou definir igualdade. Operações existentes ou extensão versionada já são o mecanismo AIR. | Separar assinatura de interação dos manifestos de nomes/codecs/tipos; remover `ContractId` dessas formas. Não inventar normalização ou igualdade no migration adapter. |
| Assinatura única `incomplete`, `signatureGaps` e sujeitos por `ContractId` | Inventários separados de parâmetros/resultados, posição/modo/TypeRef e restante próprios; sujeitos externos identificados por `invoke`. | Tipo ausente não é aridade ausente; o consumidor precisa conservar ambos. Compartilhar contrato não une sites ou ativações. | Reestruturar Signature e sujeitos de prova; conservar lacunas de modo/vínculo, posições e seus owners. Instanciar fatos antigos por site sem alargar escopo. |
| `return.entryScope` | Não é campo do núcleo. Retorno segue a entrada da ativação corrente. | Produtor deve sustentar compatibilidade das execuções; lista escolhida por Validator não pode selecionar caminhos para tornar operação válida. | Remover seletor do modelo. Se compatibilidade de múltiplas entradas não for decidida, registrar limite; não implementar análise de CFG neste trabalho de modelo. |
| `InvocationOutcomes` = envelope genérico | Distinguir contratos, unicidade e alternativas. Envelope inclui salto, saída da unidade e continuação de extensão comum. | AIR 05/06 exige comportamentos diferentes; fallback de `copy_bytes` precisa continuar sem inventar label/terminador. | Separar validação/modelo de outcomes e envelope; cobrir `continue`, `jump`, `return`, restantes e efeitos por outcome, sem contar fallback como execução extra. |
| OperandId apenas no limite ou duplicado em envelope/resultado opaco | Definir cada ocorrência uma vez sob a operação e referi-la por ID nas anotações; locais exclusivos do limite também têm definição. | I-02/I-11 exigem fechamento sem duplicar avaliação; um limite de sobrescrita pode citar local que não é argumento/resultado. | Acrescentar suporte a esses locais contratuais e eliminar duplicação de ocorrências; o adapter decide seu agrupamento físico. |
| Identidade de relações e ownership | Explicitar `ArtifactRelationId` e proprietários no modelo abstrato; `RegionId` continua StorageId de região. | Relação já tinha identidade exigida em 01, §8; codec independente precisa saber namespace/domínio. IDs de classes não fornecem essa definição. | Alinhar documentação/IDs/índices; preservar identidades existentes por correlação explícita, sem migração por texto de exibição. |
| Entrada declarada sem corpo e label opcional | Permitir ausência de label somente com corpo explicitamente indisponível; label presente sempre fecha. | AIR já admite entrada interna declarada sem corpo, mas faltava compatibilizar a regra geral de label com esse caso. Não inventar sequência. | Validar disponibilidade/assinatura/referência e distinguir ausência aplicável de lacuna, sem aceitar label faltante em corpo disponível. |
| Limites numéricos Java, versão major de capability, duas formas de bytes | Binding usa inteiros decimais arbitrários, versão declarada de capacidade e uma forma base64. | Escala/posição/contagem não têm teto de runtime na AIR; capacidade não se resume automaticamente a major. Forma física de bytes não é lista Java. | Revisar tipos limitados e serialização dos adapters; limite operacional deve ser explícito. Versão de biblioteca permanece separada. |
| Ordem de EntryState e inventários | Preservar ordem física para round-trip, sem prioridade de seeds, casos ou execução intersequência. | AIR 03 exige consistência de condições sobrepostas; lista não resolve contradição por “último vence”. | Manter ordem sem inferir precedência e testar contradições/aliases. |
| Eliminação por `PremiseId` genérico | Regra e origem de eliminação permanecem evidência do fato; não exigir nova asserção safety para representá-las. | PROD-02/I-29 já exigem eliminação justificada, não uma classe específica de premissa. | Preservar justificativa/proveniência; não converter texto de regra em semântica nova nem apagar ocorrência sem justificativa. |
| Proveniência apenas por linha/coluna | Codificar também offsets com unidade explícita, já admitidos em AIR 06. | O produtor pode conhecer apenas offsets; o consumidor deve preservar essa evidência sem inventar coordenadas. | Ampliar localização de origem/adapter; não tratar ausência de linha como origem totalmente indisponível. |
| Apêndice Java como catálogo | Substituir por catálogo físico independente, com referências aos normativos AIR. | DTOs, enums de runtime e nomes de classes não definem o modelo oficial. | Atualizar catálogo da implementação após reconciliar o modelo; manter JSON nos adapters externos. |

## Avaliação explícita das alternativas

**Contracts.** A semântica de autoridade contratual já existe; o inventário top-level
não. A necessidade bilateral é materializar assinatura, limites e premissas, não
criar uma entidade compartilhada. O modelo existente de fatos por operação e
premissas identificadas é suficiente, explicitado em 04, §7.4. Adotar o inventário
exigiria novas identidades, conteúdo, fechamento, aplicabilidade e validade; não se
justifica pelo internamento Java. A alternativa foi recusada, sem deixar conteúdo
“mágico”. Um futuro catálogo opcional exigirá proposta/negociação/versionamento
próprios, sem redefinir este `ContractRef`.

**ResourceTarget.** A necessidade de observar recurso e target já existe; a quarta
variante é atalho. As três formas AIR preservam o significado, inclusive nome
calculado em seu ponto. Incorporar ResourceId como execução mudaria o fechamento
e a interpretação dos targets. A solução é usar formas existentes, mantendo a
identidade de declaração e todas as evidências de uso.

**DisjointStorage.** A semântica de separação não foi criada pelo Validator.
Há produtor, consumidor e oráculo explícitos anteriores ao binding. Alternativas,
aliases e nomes distintos não codificam a relação entre bases; intervalos precisos
já resolvem o caso dentro da mesma base. Formalizar uma premissa mínima é necessário
para tornar essa garantia observável sem texto livre. O escopo Java não foi adotado:
a nova forma tem validade universal definida na AIR e não expressa condição dinâmica.
Ela restringe formas/obrigações do snapshot e exige revalidação, mas não introduz
análise de aliases ou resultados de storage na publicação.

**SafetyAssertion.** Pureza/validade de slice/codec são precondições do produtor;
um teste pode verificar parte da evidência. `CHOICE_REMAINDER_DOMAIN` sobrepõe a
representação de domínio da escolha inteira; `EXTENSION_EQUALITY_DEFINED` tenta
substituir manifesto por booleano. Não foi demonstrada necessidade bilateral desses
tokens. Sua remoção não elimina precondições nem proíbe prova externa: certificados
permanecem evidência separada. Uma linguagem semântica de provas seria extensão
futura com sujeitos, semântica e oráculos próprios, não campo do transporte.

## Versionamento e aceitação

A política anterior já registra o fechamento de 2.0.0 antes da publicação, em
09, §5.1. A revisão integra essa edição e conserva AIR 2.0.0/perfis @2,
**com mudanças normativas explicitamente reconhecidas** em 09, §5.2. O fato de
não haver tags reforça o contexto, mas não substitui aquela condição normativa.
Não se usa existência de `air-java` ou aprovação de JSON como estabilização da AIR.

Se a 2.0.0 estivesse estabilizada, alterar formas obrigatórias de identidade,
assinatura, premissa e validade exigiria major; não patch editorial ou minor de
metadados. Capacidade opcional negociada é alternativa possível para propostas
futuras, não justificativa para esconder quebra do núcleo. JSON 1.0.0 continua
candidata DRAFT; bytes da versão candidata anterior não têm compatibilidade prometida.

## Handoff de implementação e evidência

O próximo trabalho em `air-java` deve primeiro alinhar modelo, traversal, índices,
resolução de tipos/provas e validação às decisões acima; depois atualizar fixtures,
catálogo, status e snapshot normativo fixado. A migração deve preservar todo conteúdo
contratual nos usos, lacunas, origens, sujeitos/sites e fatos de separação sustentados;
contradição ou garantia insuficiente requer revisão/abstração explícita, não default.

I-55 a I-61 e O-86 a O-91 registram casos positivos, negativos, parciais e limites;
os perfis exigem seus sub-requisitos correspondentes. O-88-SCALAR registra a
observação de separação entre células, e O-88-REGION a separação por intervalos;
os demais novos sub-requisitos estruturais não exigem
produzir CFG, efeitos, RD ou valores. X-41 demonstra o contrato externo materializado;
as convenções existentes foram reconciliadas, inclusive C-PURE e sementes de entrada.

A verificação documental deste PR inclui whitespace/diff, links e âncoras locais,
referências a invariantes/oráculos/exemplos, integridade das tabelas e parsing dos
blocos JSON literais. Não existe suite executável de codec neste repositório.
Conformidade semântica de implementação e round-trip interoperável ainda precisam
ser demonstrados; nenhuma verificação documental é anunciada como essa prova.
