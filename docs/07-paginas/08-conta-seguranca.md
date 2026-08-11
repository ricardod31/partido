# 07/08 — Conta, chaves e segurança (SEG)

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-11 |
| **Depende de** | [07 — Páginas (índice)](README.md), [00-design-system](00-design-system.md), [03 — Cripto §3, §9–10](../03-arquitetura-criptografica.md), [06 — Ameaças](../06-modelo-de-ameacas.md) |
| **Público** | designers e engenheiros ([técnico]) com seções [conceitual] |

> A área transversal que cuida da identidade em repouso, das chaves, da rede e — o mais importante — da **honestidade consolidada**: o painel de Exposição (P-SEG-06) é a tabela "protege / não protege" do [doc 06 §5](../06-modelo-de-ameacas.md) virada tela sempre-acessível (UX6). Aqui também mora a fronteira mais frágil do modelo, dita sem eufemismo: o **cliente comprometido / cadeia de suprimento** (A7) contorna todo o E2E. O gabarito de 9 pontos ([README §5](README.md)) rege cada página.

---

## P-SEG-01 — Conta e pseudônimo

1. **Objetivo.** Gerir o pseudônimo e ver — só localmente — os handles por-organismo, sem criar correlação.
2. **Quem chega.** Todo usuário.
3. **Funcionalidades.** Trocar o `pseudonimo` (mutável, único no servidor); ver o `user_id` auto-certificante (âncora estável); ver **os meus handles por-organismo** (local ao dispositivo).
4. **Dinâmica.** O `user_id` é a âncora (derivada da chave, doc 03 §3.1); o `pseudonimo` é rótulo mutável. Os **handles são por-organismo e não-vinculáveis** (ADR-0008): a UI mostra os meus só para eu poder navegar, **nunca** os expõe correlacionados a terceiros nem ao servidor, e evita colocá-los lado a lado de modo que uma captura os ligue (design system §4).
5. **Experiência e layout.** Sem "página de perfil" que agregue organismos (UX2); a lista de handles vem com aviso de que é local e sensível.
6. **Estados.** Pseudônimo em uso (rejeita); troca propagada.
7. **Restrições.** P3; UX2; ADR-0008 (handles por-organismo).
8. **Aberto.** Esquema de identicon (design system §12).

## P-SEG-02 — Chaves e rotação

1. **Objetivo.** Rotacionar a chave de cifra sem trocar de identidade, com proteção contra rollback.
2. **Quem chega.** Usuário atento à higiene de chave; após suspeita de comprometimento.
3. **Funcionalidades.** Ver as chaves (identidade Ed25519, cifra X25519 certificada); **rotacionar** a `pk_enc` (novo certificado assinado pela `sk_id`); **revogar** certificado; ver o número de versão monotônico.
4. **Dinâmica.** A `pk_enc` é **certificada** pela identidade e **rotacionável** sem trocar o `user_id` (doc 03 §3.1). Contra rollback pelo servidor: **versão monotônica** e a regra "**só a maior versão é válida**"; a distribuição apoia-se em **key transparency** (P-SEG-08) para o cliente detectar certificado obsoleto.
5. **Experiência e layout.** `KeyManager`; fingerprints em mono; `ConfirmDestructive` na revogação; explicação [conceitual] do porquê rotacionar.
6. **Estados.** Rotação em curso; certificado revogado; possível rollback detectado (alerta via key transparency).
7. **Restrições.** doc 03 §3.1 (certificação, versão monotônica, revogação); A5 (anti-rollback).
8. **Aberto.** —

## P-SEG-03 — Dispositivos

1. **Objetivo.** Gerir os aparelhos que carregam a identidade.
2. **Quem chega.** Usuário multi-dispositivo.
3. **Funcionalidades.** Listar dispositivos; emparelhar novo (P-ON-10); **futuro:** sub-chaves por dispositivo, certificadas pela `sk_id` e **revogáveis individualmente**.
4. **Dinâmica.** MVP: a mesma seed nos aparelhos do usuário (transferência manual). **Futuro** (doc 03 §3.3): sub-chaves por dispositivo limitam o dano de A4 (apreensão de um aparelho) e permitem revogar só aquele dispositivo — inclusive útil contra a **entrega compelida de chave** (A4, coação legal).
5. **Experiência e layout.** `DeviceList`; a funcionalidade de sub-chave aparece como "em breve" no MVP, com o porquê.
6. **Estados.** Um dispositivo; múltiplos; sub-chave revogada (futuro).
7. **Restrições.** doc 03 §3.3; A4.
8. **Aberto.** Sub-chaves por dispositivo (doc 03 §3.3/§10).

## P-SEG-04 — Segurança de rede

1. **Objetivo.** Controlar o anonimato de rede e a retenção local — o *fail-closed* em detalhe.
2. **Quem chega.** Todo usuário; revisitado após o onboarding (P-ON-06).
3. **Funcionalidades.** Estado do Tor/onion; ligar/desligar **fail-closed** (padrão ligado); *bridges*; **retenção local de histórico** (não reter / reter por prazo curto).
4. **Dinâmica.** Sem canal anônimo confirmado, operações sensíveis são recusadas (UX7, doc 06 §8.2). A opção de **não reter histórico local** (ou por prazo curto) limita o dano de A4 (dispositivo comprometido).
5. **Experiência e layout.** Espelha a `NetworkStatusBar`; `HonestyCallout` sobre o custo/benefício de cada opção; `bridges` para redes que bloqueiam Tor.
6. **Estados.** Tor ativo/bloqueado; fail-closed on/off (desligar exige confirmação explícita e um aviso, pois reduz proteção); retenção configurada.
7. **Restrições.** UX7; doc 06 §8.2/A1/A4; doc 03 §9.
8. **Aberto.** Especificação exata do comportamento sem Tor (doc 06 §9).

## P-SEG-05 — Verificação do cliente

1. **Objetivo.** Dar ao usuário meios de confiar que o app que ele roda **é** o código auditado — a raiz de confiança (A7).
2. **Quem chega.** Usuário atento; a comunidade de rebuilders.
3. **Funcionalidades.** Mostrar a versão e o hash do build; instruções de **build reprodutível** e verificação por **binary transparency** (log público de binários); auto-verificação de assinatura do cliente; canal de distribuição verificável (estilo F-Droid).
4. **Dinâmica.** **A fronteira mais importante e mais frágil do modelo** (doc 06 A7): um cliente adulterado exfiltra a seed/plaintext **antes** da cifragem, e o "servidor cego" torna-se irrelevante. A honestidade é dita sem rodeios: contra um alvo específico com adversário estatal e loja cooperante, a garantia é frágil se o próprio usuário não verifica o build — o que quase ninguém faz.
5. **Experiência e layout.** `ClientVerify`; `HonestyCallout` que **não** superpromete: *"Estas verificações reduzem, não eliminam, o risco de um app adulterado. É o ponto mais difícil de toda a segurança do sistema."*
6. **Estados.** Build verificado (reprodutibilidade conferida por rebuilders); não verificado; canal de distribuição não confiável (alerta).
7. **Restrições.** doc 06 A7 (requisito, não meta futura); doc 03 §1 (cliente é o TCB).
8. **Aberto.** Rebuilders independentes, binary transparency operacional (doc 03 §10, doc 06 §9).

## P-SEG-06 — Exposição ("o que este sistema não esconde")

1. **Objetivo.** Consolidar, sempre acessível e contextual, o que o sistema **protege e não protege** — o componente-assinatura de honestidade (UX6).
2. **Quem chega.** Qualquer usuário, de qualquer tela, pelo `ExposureButton` do shell.
3. **Funcionalidades.** Listar, no contexto atual, o que fica **protegido** (conteúdo E2E) e o que fica **exposto/não protegido** (grafo de filiação e papel ao servidor até as credenciais anônimas plenas; metadados de participação; IP sem Tor; coação; dispositivo aberto; cliente adulterado A7; push — evitado). Aprofundar em cada item (D5).
4. **Dinâmica.** É a tabela "protege / não protege" do [doc 06 §5](../06-modelo-de-ameacas.md) transformada em componente vivo — e **contextual**: numa tela de voto, destaca coação; numa de finanças, o regime; num organismo, o metadado de participação. Cada estado usa a linguagem de confiança (design system §2).
5. **Experiência e layout.** `ExposurePanel` (folha); colunas protege/não-protege; ícones+rótulos (nunca só cor). Sem alarme, sem falsa garantia (UX6/D6).
6. **Estados.** Visão geral vs. contextual (adapta ao compartimento/ação atual).
7. **Restrições.** UX6; doc 06 §5 (fonte); D3/D6.
8. **Aberto.** Acompanha o fechamento das lacunas do doc 06 (credenciais anônimas, push, key transparency) — o painel deve refletir o estado real, atualizando-se conforme cada item sai de "em aberto".

## P-SEG-07 — Coação e negação plausível

1. **Objetivo.** Oferecer defesas contra a **entrega compelida** de chave/passphrase (coação legal).
2. **Quem chega.** Usuário em jurisdição com *key-disclosure laws*; contexto de risco elevado.
3. **Funcionalidades (futuro).** **Passphrase de coação** (revela um estado inócuo em vez do real); negação plausível; combina com sub-chaves revogáveis (P-SEG-03).
4. **Dinâmica.** Em jurisdições com *key-disclosure laws*, o usuário pode ser **compelido** a entregar a passphrase (doc 06 A4). Estas defesas são **evolução declarada**, não MVP (doc 03 §10) — a UI não promete o que ainda não existe.
5. **Experiência e layout.** `CoercionSettings` marcada como futura; `HonestyCallout`: *"Nenhum software protege totalmente contra coação física ou legal; estas defesas reduzem, não eliminam, o risco."*
6. **Estados.** Indisponível no MVP (explica o roadmap).
7. **Restrições.** doc 06 A4; doc 03 §10; UX6 (não superprometer).
8. **Aberto.** Passphrase de coação / negação plausível (doc 03 §10, doc 06 §9).

## P-SEG-08 — Verificação de chaves de terceiros (key transparency)

1. **Objetivo.** Detectar se o servidor serve uma chave obsoleta/falsa de outra pessoa (ataque de diretório).
2. **Quem chega.** Usuário/organismo; em geral, transparente (roda em segundo plano).
3. **Funcionalidades (near/futuro).** Consultar o **log público append-only de chaves** (estilo CONIKS); detectar certificado obsoleto ou substituição indevida; sinalizar inconsistência.
4. **Dinâmica.** Complementa a defesa criptográfica primária do MLS (confirmation_tag) — a conferência humana de fingerprint **não** é a defesa principal (doc 06 A5). Key transparency permite ao cliente **detectar** manipulação de chave sem depender do usuário conferir "safety numbers".
5. **Experiência e layout.** Majoritariamente invisível; só aparece como **alerta** quando há inconsistência. Não pede conferência manual rotineira (que a experiência mostra não acontecer — doc 06 A5).
6. **Estados.** Consistente (silencioso); inconsistência detectada (alerta forte).
7. **Restrições.** doc 03 §3.1; doc 06 A5 (defesa criptográfica, não comportamental).
8. **Aberto.** Registro verificável de chaves — evolução (doc 06 §9).

## Decisões em aberto da área

- **Build reprodutível / binary transparency operacionais** (P-SEG-05): rebuilders, log de binários — a fronteira A7.
- **Sub-chaves por dispositivo** (P-SEG-03) e **passphrase de coação** (P-SEG-07): doc 03 §3.3/§10.
- **Key transparency** (P-SEG-08): registro verificável de chaves (doc 06 §9).
- **Atualização viva do painel de Exposição** (P-SEG-06) conforme as lacunas do doc 06 se fecham.

## Referências

- [doc 03 §3, §9–10](../03-arquitetura-criptografica.md); [doc 06](../06-modelo-de-ameacas.md) (todo, sobretudo §5 e A7).
- [Design system](00-design-system.md) — `KeyManager`, `DeviceList`, `ClientVerify`, `ExposurePanel`, `TrustChip`, `CoercionSettings`.
