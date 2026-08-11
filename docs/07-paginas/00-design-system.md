# 07/00 — Design system

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-11 |
| **Depende de** | [07 — Páginas (índice)](README.md), [00 — Visão](../00-visao.md), [06 — Modelo de ameaças](../06-modelo-de-ameacas.md) |
| **Alimenta** | todos os documentos de área (A–I) |
| **Público** | designers e engenheiros ([técnico]) com seções [conceitual] |

> O design system é o vocabulário visual e interativo comum a todas as páginas. Ele não é decoração: cada escolha aqui **serve um princípio ou um adversário** dos docs 00/06. Três coisas o distinguem de um design system comum: (1) uma **linguagem de confiança e exposição** de primeira classe (UX6); (2) **compartimentação** como propriedade visual, não só lógica (UX1); (3) **pseudonimato sem correlação** embutido no jeito de mostrar pessoas (UX2). Tokens têm valores concretos para serem implementáveis; nomes de token são estáveis e citáveis.

---

## 1. Princípios de design [conceitual]

Os oito compromissos de produto (UX1–UX8, [README §1](README.md)) descem para sete princípios de design:

- **D1 — Sobriedade, não engajamento.** Densidade de ferramenta de trabalho. Sem *infinite scroll*, sem contadores de curtida/visualização, sem animação recompensadora, sem cor gritante por padrão. A atenção é do usuário, não do produto.
- **D2 — O compartimento é visível e íntegro.** Toda tela declara a qual organismo pertence; nenhuma tela mistura dois. Trocar de compartimento é um gesto explícito e sentido (§3).
- **D3 — A confiança é mostrada, não presumida.** O que está cifrado, o que é metadado exposto, o que não é protegido — cada um tem forma visual própria e **sempre acompanhada de texto e ícone** (nunca só cor). (§2)
- **D4 — Pseudonimato por construção.** Pessoas aparecem por handle local ao organismo + identicon; a interface não oferece caminho para correlacionar identidades entre compartimentos. (§4)
- **D5 — Divulgação progressiva.** A superfície fala [conceitual]; o [técnico] (epoch, prova, fingerprint) está a um toque, nunca imposto.
- **D6 — Honestidade e ausência de *dark patterns*.** Avisos são verdadeiros e proporcionais; os poucos avisos **não-dispensáveis** existem só onde o modelo de ameaças exige (voto *fail-closed*, modo anônimo ilegal). Nunca se induz uma ação por atrito assimétrico.
- **D7 — Resiliência primeiro.** Funciona em Android de entrada, offline, sobre Tor, sem buscar recurso externo (sem CDN — é desempenho **e** opsec). Sem telemetria, sem analytics.

## 2. Linguagem de confiança e exposição [técnico]

O elemento-assinatura do produto (UX6, D3). Quatro **estados de confiança**, cada um com cor, ícone e rótulo — os três juntos, sempre (acessibilidade: cor nunca é o único portador).

| Estado | Token de cor | Ícone | Rótulo (exemplo) | Significa |
|---|---|---|---|---|
| **Protegido** | `--trust-protected` | cadeado fechado | "Cifrado ponta a ponta" | Conteúdo E2E (MLS); o servidor não lê |
| **Exposto** | `--trust-exposed` | olho | "O servidor vê isto" | Metadado visível ao servidor (organismo de destino, horário, tamanho) |
| **Não protegido** | `--trust-unprotected` | escudo vazado | "Isto não te protege contra…" | Fora do alcance do desenho (coação, dispositivo aberto, análise global) |
| **Anônimo** | `--trust-anon` | máscara | "Autorizado sem te identificar" | Credencial anônima ativa (BBS+/KVAC): autoriza sem revelar quem |

Três componentes materializam esses estados:

- **TrustChip** — etiqueta compacta inline (ex.: no cabeçalho de um mural, "Cifrado" + "O servidor vê: destino, horário"). Tocável → abre a divulgação de metadados.
- **HonestyCallout** — bloco curto no ponto de uso, antes de uma ação sensível. Ex., antes do voto secreto: *"O voto é secreto perante o sistema. Ele **não** protege se alguém te obriga a mostrar como votou."* (doc 06 §5).
- **ExposurePanel** (P-SEG-06) — a visão consolidada "o que este sistema **não** esconde, aqui e agora", montada a partir do doc 06 §5. Acessível pelo shell (§6) de qualquer tela.

**Regra de ouro (D3):** nenhuma promessa de segurança aparece sem sua contrapartida. Onde a UI diz "cifrado", ela também diz, ali, o que **fica exposto**. É a tabela "protege / não protege" do doc 06 §5 transformada em componente.

## 3. Compartimentação visual [técnico]

Cada organismo tem uma **cor de identidade** determinística (derivada do seu `id`, escolhida da paleta de compartimentos §5.3 — nunca das cores semânticas). Ela pinta a **espinha do compartimento**: uma faixa/borda persistente que emoldura toda tela pertencente àquele organismo, mais o cabeçalho do compartimento (tipo do organismo, nome interno, época).

- **Ao entrar** num compartimento, a espinha assume a cor daquele organismo e o cabeçalho se identifica.
- **Trocar de compartimento** (P-NAV-03) é uma transição de tela explícita (não uma aba silenciosa): a espinha muda de cor com um movimento curto e perceptível, sinalizando "você cruzou uma fronteira de segurança". (Respeita `prefers-reduced-motion`: vira um *cross-fade* sem deslocamento.)
- **Nada de dois compartimentos na mesma tela.** Não há *split view* de organismos; não há menção cruzada. A cor é auxílio, não a garantia — a garantia é topológica (a UI simplesmente não carrega conteúdo de outro organismo).

## 4. Pseudonimato na interface [técnico]

Como o produto mostra pessoas (UX2, D4, [ADR-0008](../decisoes/adr-0008-mls-e-credenciais-anonimas.md)):

- **Handle por-organismo.** A unidade de identidade na UI é o **handle local ao organismo** (não um `user_id` global). "Camarada Pórtico" na Célula A e "Camarada Estopim" na Comissão de Finanças **podem** ser a mesma pessoa — e a interface **jamais** revela ou insinua isso. Não há página de perfil que agregue os organismos de uma pessoa.
- **Avatar = identicon do handle.** Gerado deterministicamente do handle (não é foto, não há upload de imagem de identidade civil). Muda com o handle; é local ao compartimento.
- **Autoria tem dois horizontes** (doc 03 §6.1): **entre membros** de um organismo, mensagens e propostas são atribuídas ao handle (a deliberação exige saber quem falou); **perante o servidor**, a autoria é uma credencial anônima (o servidor não aprende quem). O voto **secreto** é não-atribuível mesmo entre membros (§ DEL). O voto **aberto** é nominal por desenho.
- **O próprio usuário** vê, só no seu dispositivo, quais handles são seus (para poder trocar de compartimento). Essa correlação **nunca** transita para o servidor nem para outros membros, e a UI evita colocá-los lado a lado de modo que uma captura de tela os vincule.

## 5. Tokens [técnico]

Dark-first (opsec, OLED, perfil baixo). Tema claro espelha os papéis; "sistema" segue o SO. Sem web-fonts remotas — só stacks de fonte do sistema.

### 5.1 Cor — tema escuro (padrão)

| Token | Valor | Uso |
|---|---|---|
| `--bg-0` | `#0E1116` | Fundo mais profundo (app) |
| `--bg-1` | `#151A21` | Superfície de conteúdo |
| `--bg-2` | `#1C222B` | Superfície elevada (cartão, folha) |
| `--border` | `#242C37` | Bordas e divisórias |
| `--ink-0` | `#E8EDF2` | Texto primário |
| `--ink-1` | `#AEB8C4` | Texto secundário |
| `--ink-2` | `#6B7684` | Texto desabilitado/hint |
| `--accent` | `#5B8DEF` | Ação primária, foco, links |
| `--accent-ink` | `#0E1116` | Texto sobre `--accent` |

### 5.2 Cor — semânticas de confiança (compartilhadas entre temas)

| Token | Valor (escuro / claro) | Papel |
|---|---|---|
| `--trust-protected` | `#3FB27F` / `#1F7A54` | Protegido (cifrado) — também "sucesso" |
| `--trust-exposed` | `#E0A33E` / `#9A6B12` | Exposto (metadado) — também "atenção" |
| `--trust-unprotected` | `#E5686A` / `#B23033` | Não protegido / destrutivo — também "perigo" |
| `--trust-anon` | `#9B7BE0` / `#6A48B8` | Credencial anônima ativa |

Contraste alvo: texto ≥ 4.5:1, ícones/estados ≥ 3:1 (WCAG AA). As cores de confiança **nunca** aparecem sozinhas (sempre ícone+rótulo, §2).

### 5.3 Cor — paleta de compartimentos (identidade de organismo, §3)

Oito matizes distintos e acessíveis, **sem** verde/âmbar/vermelho puros (reservados às semânticas): `#6E8AD6` (índigo), `#3E9CA8` (teal), `#9A6DB0` (ameixa), `#5E9E77` (musgo — distinto do verde-confiança), `#C08457` (barro), `#7C8A99` (aço), `#B08AC0` (lavanda), `#4FA0C4` (ciano). Atribuídos deterministicamente por `id` do organismo.

### 5.4 Tipografia

| Papel | Stack | Uso |
|---|---|---|
| UI (workhorse) | `-apple-system, "Segoe UI", Roboto, "Noto Sans", system-ui, sans-serif` | Interface geral |
| Leitura (long-form) | `"Iowan Old Style", "Noto Serif", Georgia, serif` | Jornal e publicações (P-JOR-03) — sinaliza "modo leitura" |
| Mono | `ui-monospace, "JetBrains Mono", "Roboto Mono", monospace` | Fingerprints, chaves, `id`, séries de voucher — **verificação exige mono** |

Escala (px / line-height): 12/16, 14/20, 16/24 (corpo), 18/26, 20/28, 24/32, 30/38, 36/44. Peso: 400 corpo, 600 títulos, 500 rótulos.

### 5.5 Espaçamento, raio, elevação, movimento

- **Espaço** (base 4): 4, 8, 12, 16, 24, 32, 48, 64.
- **Raio:** 6 (controles), 10 (cartões), 16 (folhas), full (chips/avatar).
- **Elevação:** mínima — hierarquia por `--bg-*` e `--border`, não por sombra pesada (D1); folhas/modais usam uma sombra sutil única.
- **Movimento:** rápido e funcional (120–200 ms); a transição de compartimento (§3) é a única "sentida". `prefers-reduced-motion` remove deslocamentos.

## 6. O shell da aplicação [técnico]

O chrome persistente (README §3.2). Layout responsivo: coluna única no celular (alvo primário), coluna lateral + conteúdo no desktop.

- **CompartmentHeader** — cor de identidade (§3), tipo+nome do organismo, EpochIndicator.
- **CompartmentSwitcher** — troca deliberada de organismo (P-NAV-03); lista os meus organismos com sua cor.
- **NetworkStatusBar** — o coração do *fail-closed* (UX7): Tor on/off (onion conectado?), LockIndicator (chave travada/destravada), e alertas de época (rekey). Quando Tor cai, esta barra muda de estado e as ações sensíveis passam a bloquear (§7 FailClosed).
- **ExposureButton** — sempre presente; abre o ExposurePanel (§2).
- **LocalNoticeCenter** — avisos **locais** (P-NAV-05), por *polling* sobre Tor; nunca push (A7, doc 06 I2).

## 7. Estados transversais [técnico]

Todo componente que carrega dados implementa este conjunto. Não são exceções — são parte do contrato.

| Estado | Aparência | Regra |
|---|---|---|
| **Carregando** | Skeleton (não *spinner* infinito) | Preserva layout; sem salto |
| **Vazio** | EmptyState com próximo passo | Explica o porquê (ex.: "need-to-know: você não vê o histórico anterior à sua entrada") |
| **Offline** | Faixa discreta; ações enfileiradas | Local-first; sincroniza ao voltar; nunca perde rascunho |
| **Fail-closed** | **FailClosedBlocker** — overlay não-dispensável | Sem canal anônimo, recusa a ação sensível (voto, sobretudo) com explicação; nunca cai para clearnet (UX7, doc 06 §8.2) |
| **Pendente de admissão** | Estado de espera (P-ON-07) | Conta existe, mas a célula ainda não admitiu (Commit `Add`) |
| **Rekey / época mudando** | EpochIndicator ativo; envio breve travado | Transição de `Commit` (I9); explica em [conceitual] ("a composição do grupo mudou") |
| **Revogado / expirado** | MandateCard esmaecido + motivo | Mandato terminou (I4): `expirado` ou `revogado`, com `data_fim_efetiva` |
| **Sancionado** | Estado do membro (censura/afastado/desligado) | Só como desfecho de deliberação (I11); mostra a resolução de origem |
| **Erro** | Mensagem acionável, sem jargão de stack | Nunca expõe detalhe que vaze metadado |

## 8. Catálogo de componentes [técnico]

Organizados por família. Cada um é citável pelo nome; os documentos de área referenciam-nos.

**Shell:** `AppShell`, `CompartmentHeader`, `CompartmentSwitcher`, `NetworkStatusBar`, `LockIndicator`, `EpochIndicator`, `ExposureButton`, `LocalNoticeCenter`.

**Identidade:** `HandleChip` (identicon+handle, escopo-local), `PseudonymAvatar` (identicon), `RoleBadge` (papel + proveniência do mandato), `MandateCard` (destino, prazo, recall, fonte).

**Confiança/honestidade:** `TrustChip`, `HonestyCallout`, `ExposurePanel`, `FailClosedBlocker`, `MetadataDisclosure`.

**Deliberação:** `DeliberationStepper` (máquina de estados discussão→…→resolução), `ProposalCard`, `AmendmentThread`, `TendencyGroup` (plataformas/tendências, doc 01 C.1), `OpenBallot` (commit-reveal), `SecretBallot` (blinding), `UrnDeposit` (Tor + mistura), `BulletinBoardPanel` (nº credenciais vs censo, I12), `QuorumMeter`, `ResolutionAta` (assinada, imutável, com elo `substitui`).

**Jornal:** `PublicationReader` (com selo de assinatura verificada do órgão editor), `PublicationEditor`, `EditorialQueue`, `CorrespondenceComposer` (fluxo ascendente), `ScopeSelector` (`publico`/`interno_organizacao`/`interno_organismo`).

**Finanças:** `AggregatePanel` (só agregados — nunca pessoa→valor), `CotizationStatus`, `VoucherIssue`, `VoucherRedeem`, `AccountabilityReport` (coassinatura), `RegimeGate` (bloqueia anônimo se partido registrado).

**Federação:** `FrontCard`, `AgreementCoSign` (chave da organização), `FingerprintVerify` (fora da banda), `DelegateAttestation`.

**Segurança:** `KeyManager`, `DeviceList`, `ClientVerify` (build reprodutível), `RecoveryFlow` (social), `CoercionSettings` (futuro).

**Cerimônias (full-screen, multi-passo, para ações sensíveis):** `Ceremony` genérico usado por backup mnemônico (P-ON-04), voto secreto (P-DEL-06/07), coassinatura de acordo (P-FED-03), verificação de fingerprint (P-FED-03). Uma cerimônia é focada (sem chrome distraente), sequencial e confirmável, e registra explicitamente o que está prestes a acontecer.

**Primitivos:** `Button` (primário/secundário/perigo), `Sheet`, `Modal`, `Stepper`, `Toast` (local), `EmptyState`, `Skeleton`, `ConfirmDestructive`.

## 9. Iconografia [técnico]

Traço consistente (2px), desenhados inline (SVG no bundle — sem icon-font remota, D7). Conjuntos: **tipos de organismo** (célula, comitê, comissão, fração, congresso, direção — distintos entre si), **papéis** (secretário, tesoureiro, agitprop, delegado), **estados de confiança** (§2), **ações** (propor, votar, publicar, corresponder, cotizar). O ícone de estado de confiança nunca é reutilizado para outra semântica.

## 10. Acessibilidade, i18n e desempenho [técnico]

- **Contraste** WCAG AA (§5.2); **cor nunca sozinha** (D3) — todo estado tem ícone+texto.
- **Leitor de tela:** papéis semânticos; TrustChip e estados têm alternativa textual completa; a espinha de compartimento é anunciada ("Compartimento: Célula A").
- **Toque/teclado:** alvos ≥ 44 px; foco visível; navegação por teclado completa no desktop.
- **i18n:** pt-BR primeiro; strings externalizadas; pronto para outras línguas e RTL. Nada de texto embutido em imagem.
- **Desempenho e opsec:** alvo Android de entrada; *offline-first* (armazenamento local cifrado); **zero recurso externo** (fontes do sistema, ícones inline) — o que também evita vazamento de rede a CDNs; *polling* consciente de bateria; **sem telemetria/analytics** (D7).
- **Movimento reduzido:** honra `prefers-reduced-motion`, inclusive na transição de compartimento (§3).

## 11. Padrões de microcopy [conceitual]

- **Duas camadas:** o texto padrão é [conceitual]; o detalhe [técnico] abre por divulgação progressiva (D5).
- **Voz da honestidade** (UX6): clara, proporcional, sem alarme nem falsa garantia. Diz o que protege **e** o que não. Ex.: *"Ninguém no servidor lê esta mensagem. O servidor registra que houve uma mensagem para esta célula, e quando."*
- **Avisos não-dispensáveis** só onde o modelo de ameaças exige: voto sem canal anônimo (bloqueia), modo anônimo sob regime de partido registrado (bloqueia, doc 05 §2). Todo o resto é dispensável e reversível.
- **Sem *dark patterns* (D6):** nenhuma ação destrutiva ou irreversível (desligamento, rotação de chave, dissolução) sem `ConfirmDestructive` claro; nenhuma pré-seleção que empurre para menos privacidade.

## 12. Decisões em aberto

- **Tokens finais e marca.** Os valores de cor/tipografia são um ponto de partida sóbrio; a identidade visual definitiva (e o nome do projeto, doc 00 §6) é decisão posterior.
- **Densidade adaptativa.** Perfil "compacto" vs "confortável" por preferência — a fixar após testes com organizações reais.
- **Identicon.** Esquema determinístico concreto (evitar colisões visuais e conotações indesejadas) — a especificar.
- **Estratégia de ícones.** Biblioteca-base vs desenho próprio — a decidir na implementação, respeitando D7 (sem fonte remota).

## 13. Referências

- [README de páginas](README.md) (UX1–UX8, personas, IA).
- [doc 06 §5/§8](../06-modelo-de-ameacas.md) — origem da linguagem de confiança/exposição e do *fail-closed*.
- [ADR-0008](../decisoes/adr-0008-mls-e-credenciais-anonimas.md) — handles por-organismo e credencial anônima.
