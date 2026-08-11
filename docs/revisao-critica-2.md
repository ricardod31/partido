# Revisão crítica 2 — backlog consolidado de decisões pendentes

| | |
|---|---|
| **Status** | ativo (registro de pendências) |
| **Última atualização** | 2026-08-11 |
| **Origem** | 2ª rodada adversarial sobre os docs 00–06 + 3 rodadas de validação da camada de produto ([docs/07-paginas/](07-paginas/)) — 12 agentes de análise e 3 de verificação, com refutação cruzada |
| **Relação** | sucede a [revisao-critica.md](revisao-critica.md) (1ª rodada, já aplicada); alimenta os próximos ADRs e emendas dos docs 02/03/06 |
| **Público** | todos |

> Este documento é o **backlog canônico** do que está decidido-como-pendente: cada item foi confirmado por 2+ análises independentes (ou por verificação adversarial dedicada), tem dono documental (qual doc/ADR o resolve) e — quando a camada de produto depende dele — um identificador **DEP-nn** citado nas specs de página ([doc 07 §7](07-paginas/README.md)). Nada aqui é esquecimento: são decisões com forma conhecida esperando a fase certa. **As recusas registradas ao fim (§6) valem tanto quanto as pendências — impedem que uma "solução" já derrubada volte por outra porta.**

## 1. ADRs a escrever (arquitetura/domínio)

| # | ADR pendente | Resolve | DEP |
|---|---|---|---|
| 1 | **Gênese/fundação** — cerimônia de dia zero (chave da organização, estatuto inicial, célula fundadora, papéis provisórios com prazo); salvaguardas: ≥3 fundadores, convite-gênese contado/expirável, limites e visibilidade do estado provisório ("célula sem buro"), ato de fundação como raiz da cadeia de auditoria; exceção *desenhada* de I3/I6 | circularidade do dia zero (registro exige secretário → eleição → célula → criador) | DEP-06 |
| 2 | **Identidade** — matriz de identificadores (`user_id` global × pseudônimo único-no-servidor × handles por-organismo × roster do MLS-DS); o que aparece onde, para quem; identificador dos **atestados de mandato/frente** (hoje cruzam compartimentos pelo pseudônimo); normalização/unicidade de nomes (NFKC, confusables) como **norma de domínio**; avaliar se o pseudônimo global único se justifica | UX2 vira propriedade criptográfica, não só de interface | DEP-05 |
| 3 | **Mensageria entre organismos** — envio de não-membro para um grupo (ex.: chave caixa-postal do organismo, certificada pelo grupo e rotacionada por epoch); casos enumerados separadamente: correspondência à redação, correspondência à instância superior, prestação de contas (mandato e financeira), contagem do limiar do congresso extraordinário, pedido de recuperação social, atestados de credenciamento, variante **cross-servidor** (delegado externo escrevendo na sede da frente); a caixa-postal pode simplificar também a **descida** (selar por organismo-alvo, O(organismos) em vez de O(membros)) | "só o destino lê" sem mecanismo; a metade ascendente do P4 | DEP-04 (e alimenta DEP-01) |
| 4 | **I12 — terceira via** — censo congela na abertura da deliberação (padrão, já vigente); re-abertura do censo **por deliberação do próprio organismo**, só até abrir a votação; da votação à apuração, imutável. Casos de borda: readmitido não lê a discussão anterior (need-to-know); desligado (I11) durante deliberação — resolver denominador × remoção MLS | custo democrático de discussões longas sem reabrir a janela de *packing* | — |
| 5 | **Congresso × árvore/I13** — reposicionar a árvore (ORG→Congresso→CC, fiel ao doc 01 A.7 — o doc 02 §1 diverge da própria fonte) **ou** exceção tipada em I13 ("para `tipo=congresso`, escopo ⊆ descendentes da raiz"); tratar o **pai dissolvido** (congresso é temporário; re-parenting do CC não está modelado em I2); emenda de I4 já roteada ao ADR do item 6; frente = **readoção interna** (sem exceção em I13) | o órgão supremo hoje não vincula ninguém sob I13 literal | DEP-07 |
| 6 | **Mandante persistente de recall** — congresso extraordinário por limiar de X% de células como `mandante` de recall dos mandatos do CC (doc 02 §2.7 e P-MAN-06 já o desenham; falta o ADR que o doc 02 exige) | nenhum mandato fora do alcance da base | — |
| 7 | **Assinatura do organismo** — esquema coletivo concreto (FROST, RFC 9591) para resoluções, publicações do órgão editor e acordos de frente; *deniability* rescopada (o MLS assina o conteúdo → apreensão de dispositivo dá autoria; a deniability é perante o servidor) | "assinatura_do_organismo" hoje sem mecanismo | DEP-02 |
| 8 | **Sanção e devido processo** — estado "censura" no enum de membro; tipo de deliberação disciplinar; **regra de competência** (quem sanciona quem); posição do acusado no censo; notificação, defesa e **apelação à instância superior** (ADR-0009 deixou "direito de defesa?" aberto; a tradição citada no doc 01 tinha instâncias de recurso); revogação da credencial anônima na saída | P-DEL-10 sem base completa de domínio | — |
| 9 | **Saída voluntária e transferência** — emenda a I11 ("ou por ato registrado do próprio"), com limpeza de filiações/mandatos; transferência de célula-base atômica sob I1 (saída + admissão no destino, épocas dos dois lados) | hoje ninguém sai sem deliberação alheia; membros-fantasma apodrecem quórum | — |
| 10 | **Tendência/Plataforma** — entidade de agrupamento de posições na discussão (doc 01 C.1: "ou ela é acrescentada, ou a promessa é suavizada"); compõe com a pauta pré-congresso | P-DEL-03 desenha um componente sem entidade | — |
| 11 | **Recuperação social** — protocolo de re-atestação por quórum da célula (doc 06 §8.6 pede "perto do MVP"); cruza organismos (item 3) | P-ON-11 bloqueada na versão final | — |
| 12 | **Lista canônica fail-closed** — emenda ao doc 06 (§9 já registra o aberto): 3 classes (voto = recusa inegociável; operações de membro = recusa por padrão + *bridges*; leitura pública = permitida com aviso); custo de disponibilidade assumido explicitamente; sem toggle global em nenhuma superfície | DEP-10 |

## 2. Emendas de domínio (doc 02) sem ADR próprio

- **Estatuto versionado** + regra "deliberação corre sob o estatuto vigente na abertura" (interage com I8/I12); fluxo de **reforma estatutária** (deliberação de congresso) e superfície de leitura do estatuto global completo (P-ORG-06 já enumera; falta a reforma).
- **Dissolução/fusão/divisão de organismo** — as transições de `estado` (`ativo→suspenso→dissolvido`) sem dono; destino de membros (I1), filhos (I2), arquivo e saldo; a divisão obrigatória quando a célula estoura o teto **imposto** (doc 06 §8.3) precisa do remédio que hoje não existe.
- **Suplência** — promoção titular→suplente (vacância/ausência), doc 02 §7; sem ela, quórum de congresso trava ou a substituição foge do registro (I3).
- **Despesas** — o doc 05 só modela entrada; lançamentos de saída (append-only, coassinados) com A8 em dobro (o desvio clássico é na saída); alimenta P-FIN-06/07.
- **Reuniões** — a persona do secretário "convoca reuniões" sem entidade: modelar Reunião (convocação, ata) ou declarar explicitamente que é social via mural; lista de presença é metadado sensível.
- **Harmonizações de citação na fonte**: I11 é citada no doc 02 §2.2 e doc 06 A8 para *papéis* — ou o texto de I11 passa a cobrir papéis, ou as citações mudam para §2.2; "doc 06 I2" nasceu no doc 03 §9 (já corrigido) — conferir se outros documentos herdaram.

## 3. Cripto (doc 03) — pendências além dos ADRs

- **DEP-08 — envelope**: resolver a contradição interna do §6.1 (contador "por remetente" em claro × `prova_membro` anônima; §6.2(4) sugere cadeia por-feed, que colide com escrita concorrente). Direção: cadeia **por feed** + checkpoint assinado (STH) difundido via grupo MLS. Nota de revisão já inserida no doc 03; **nenhuma das duas leituras deve ser implementada** antes da resolução.
- **Comprometimento da `sk_id`** — o doc 03 §3.1 cobre rotação/revogação da chave de **cifra**; o comprometimento da chave de **identidade** não tem caminho (vizinha da recuperação social). Lacuna de desenho nova, achada na rodada 3.
- **ADR-0003 desatualizada** — o texto de Decisão ainda fala em "assinatura do remetente conferida" (pré-ADR-0008, o remetente nominal saiu do cabeçalho); passada de atualização antes de ser citada como lastro.
- **Voto** — bulletin board em **três números** (censo · emitidas · depositadas; emitidas−depositadas = "não utilizadas", agregado; credencial expirada continua em "emitidas"); **auto-verificação privada** da própria entrada no censo ("não pedi credencial mas consto" → alarme com disputa via escrutinadores); parâmetros de mistura e janelas de lote com aleatorização; conjunto de signatários **uniforme por eleição**.
- **Handshakes MLS** — KeyPackage/Welcome/Commit expõem identidade (exceção assinada do §6.3); anonimato nesses objetos não é coberto pela credencial anônima; `prova_membro` amarrada aos bytes do envelope.

## 4. Jurídico

- **LGPD** — ausente do repositório: opinião política é dado pessoal **sensível** (art. 5º, II); pseudônimo ≠ anonimização (o grafo pseudônimo re-identificável do doc 06 A3 pode fazer do operador do servidor um controlador); imutabilidade (I10, logs append-only) × direito de eliminação nunca confrontados. Análise dedicada antes da implementação.
- **Livro identificado do modo transparente** — a prestação TSE de partido registrado exige pessoa→valor com recibos; o servidor E2E recusa guardá-lo por construção; decidir onde vive (tesoureiro? sistema contábil externo? camada local exportável?) — P-FIN-03 sinaliza.

## 5. Produto/design system — próxima iteração (não bloqueiam prototipagem)

- Estratificação do MVP (~73 páginas **não** é mínimo; definir o primeiro ciclo real).
- Tokens: tema claro com valores; correção de contraste (`--ink-2` reprova 4.5:1 nas três superfícies; lavanda/ciano da paleta de compartimentos <3:1 no claro); separar cores de confiança dos papéis genéricos sucesso/perigo.
- Alta fidelidade: estados interativos/focus ring; breakpoints; anatomia dos componentes-chave; medidas da espinha; tabela única de strings canônicas de honestidade.
- Fluxo guiado de **incidente de dispositivo** (orquestra P-SEG-02 + aviso à célula + rekey).
- Página de operação do servidor: **não haverá** — runbook operacional em documento próprio, fora do produto (uma área de "admin de infra" na UI institucionalizaria o assento de operador que I6/UX3 negam); a declaração de fora-de-escopo é esta.

## 6. Recusas registradas (não reabrir sem argumento novo)

1. **Bulletin board por-entrada público** (censo com marcação credenciado/não por pessoa): recusado — tornaria a **não-participação verificável**, dando ao coator como exigir e conferir abstenção; contradiz o doc 03 §8.2 ("sem identidade"). Vale a via dos três números agregados + auto-verificação privada (§3).
2. **Toggle global de desligar o fail-closed**: recusado — doc 06 §8 ("imposta pelo sistema, não sugerida"); o dano de clearnet é coletivo (encolhe o conjunto de anonimato). A flexibilidade legítima vive nas classes da lista canônica (item 1.12), decidida no doc 06 — nunca numa preferência.
3. **Reemissão de resoluções de congresso pela direção** (como solução de I13): recusado — inverte a supremacia (a direção é eleita *pelo* congresso e viraria gatekeeper do órgão que deve obedecer). Valem as vias do item 1.5.
4. **"Assinatura limiar de RSA cega" como requisito**: recusado como especificação — não existe padrão/artefato auditado (P7); a direção é k assinaturas RFC 9474 independentes (k > n/2), registrada no item 3.
5. **Páginas de produto para operações de domínio inexistentes** (saída, transferência, dissolução, despesas, reuniões): recusado desenhá-las antes do domínio — repetiria o erro que a camada 07 evita ("não legislar domínio por página"); primeiro os itens 1.9/2, depois as telas.

## Referências

- [revisao-critica.md](revisao-critica.md) — 1ª rodada (aplicada); seus achados ainda-abertos foram absorvidos aqui.
- [docs/07-paginas/README.md §7](07-paginas/README.md) — tabela DEP-01..DEP-12 (visão da camada de produto destas pendências).
- Docs 02/03/04/06 e ADRs citados em cada item.
