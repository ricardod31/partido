# Protótipo de telas — ciclos 1–5

**Um único arquivo autocontido** ([index.html](index.html)) — abra no navegador, sem servidor, sem dependência externa (coerente com D7: zero CDN, fontes de sistema, ícones inline). É uma **bancada de demonstração**, não o produto: dados 100% fictícios, e os selos tracejados `DEP-nn` marcam desenho em aberto (registro canônico em [docs/07-paginas/README.md §7](../docs/07-paginas/README.md)).

## O que demonstra

Telas do plano de prototipagem ([doc 07 §8](../docs/07-paginas/README.md)): Panorama (regra de conteúdo dos cartões + **checklist de primeiros passos**), Mural (TrustChip → MetadataDisclosure, resolução descida, need-to-know), Seletor com **transição de espinha**, Deliberações (stepper, censo congelado na abertura — I12), Discussão com tendências, **cerimônia de voto secreto** (coleta k-de-n, janela de mistura, estado re-entrante — e o **fail-closed ao vivo**: derrube o Tor na bancada durante o depósito), Apuração com bulletin board em 3 números, Resolução (ata serifada, `substitui`, assinatura do organismo), Membros & papéis (proveniência, sem botão de remover), Emissão de convites (log intra-compartimento, chip condicionado), Cotização + voucher (**agora com valor e como pagar**), Painel de Exposição contextual (com o infiltrado A2), o onboarding F1 completo (convite → pseudônimo → **senha de uso diário → chave-mestra de papel** → Tor → pendente de admissão), e a **camada de aprendizado**: cartilha com lições curtas, **ensaio de votação** e glossário sob toque.

## Ciclos

- **Ciclo 1** — bancada base: telas prioritárias, cerimônia de voto, onboarding, TrustChip/Exposição.
- **Ciclo 2** — cobertura e coerência: mais telas transversais, dados por compartimento consolidados.
- **Ciclo 3** — resposta à auditoria multi-agente (usabilidade, acessibilidade, cobertura, adversarial). Quatro frentes:
  1. **Integridade** — dados isolados por compartimento (nada da Célula vaza no Congresso; cada sala tem seu handle, sua fechadura, seus membros); **gate offline do voto** (a cerimônia trava se a rede protegida cair, no início *e* no meio — exige Tor **e** conexão durante o depósito); **censo coerente** (6 membros = 6 aptos a votar; candidatos presentes no quadro); fail-closed também na coleta k-de-n.
  2. **Simplicidade e tradução de jargão** — "época" vira **"fechadura"** em toda a interface; CTAs em linguagem comum; **cota com valor (R$ 30) e um "como pagar" 1-2-3**; opção de **PIN/biometria** no onboarding; termos técnicos viram **GlossaryTerm** (sublinhado pontilhado → explicação sob toque).
  3. **Camada de aprendizado** (pedido do usuário — "tutoriais para ensinar") — **StarterChecklist** no Panorama (4 primeiros passos que se marcam ao concluir); **Cartilha** com lições L1–L4 (por que nomes por sala, chave-mestra × senha, voto secreto e o que ele não protege, o que o sistema não esconde); **Ensaio de votação** (`PracticeFrame`: pratica todos os passos do voto **sem cédula real, sem rede, nada vale**), acessível do detalhe da deliberação e da cartilha; glossário navegável.
  4. **Acessibilidade** (WCAG 2.2 AA) — `lang="pt-BR"`, regiões `aria-live` (educadas e assertivas) anunciando cada transição, **Esc** fecha folhas e aborta cerimônia, gerência de foco em diálogos e no bloqueio de rede, `aria-pressed`/`aria-current` nos controles, alvos de 44 px, borda de campo visível.

- **Ciclo 4** — fechamento de cobertura:
  1. **Emendas (P-DEL-04)** — deliberação em fase de emendas com **versão de texto** (v1 guardada, v2 em vigor, trecho emendado marcado), emenda incorporada × apresentada, e as duas regras ditas na tela: emenda aprovada **reabre o debate no texto mudado**, e emendar **não reabre a lista de votantes** (I12).
  2. **Regime financeiro (P-FIN-08) × PIX (P-FIN-03)** — a bancada alterna a organização entre *movimento/associação* e *partido registrado*: no segundo, o vale em espécie **tranca por lei** (9.096/95), o PIX identificado assume com aviso do que expõe, e a folha de metadados e o Painel de Exposição **ramificam pelo regime** (não prometem anonimato que a lei proíbe).
  3. **Minhas salas + árvore (P-NAV-02 / P-ORG-07)** — lista transversal das salas com handle e fechadura por sala (com o custo do agregador dito: "uma captura entrega seu mapa"), e a árvore da organização com a honestidade A3: **a estrutura fica no servidor; o conteúdo, não** — salas alheias aparecem como existência, nunca composição.
  4. **Acesso (P-ON-08/09/10)** — destravar (senha do dia × chave-mestra, com o limite dito), recuperar pela chave-mestra (o que volta: identidade; o que não volta: histórico local e o passado de cada sala; ninguém é avisado), e emparelhar aparelho novo (presencial, QR + conferência de números, "quem filma o código vira você"; desparear à distância ainda não existe — dito).
- **Ciclo 5 — alta fidelidade** (aparência candidata a final, a polir; dados e criptografia seguem mockados):
  1. **Visual fixado** — tokens refinados (superfícies em 4 níveis, tintas de confiança com fundo, `color-mix` da espinha no cabeçalho), **estados interativos completos** (hover/pressed/focus/disabled), escala de elevação/z (espinha 40 < cerimônia 45 < folhas 50 < bloqueio 60), movimento 140/240 ms com `reduced-motion` zerando, PracticeFrame tracejado + faixa listrada. As baixas correspondentes estão no [DS §12](../docs/07-paginas/00-design-system.md).
  2. **Fidelidade funcional** — o mural **publica de verdade** (mensagem sua entra no feed, com bolha no tom da sala; sem conexão, entra **na fila** com selo e envia quando a conexão volta); **propor cria a decisão** na lista (com debate aberto e censo congelado); **emendas entram** na lista com numeração; **convites nascem** com código próprio e aparecem em "Convites"; avisos se marcam **lidos** (a bolinha some).
  3. **Últimas telas do inventário** — **Boas-vindas (P-ON-01)**: os três caminhos + "fundar" trancado dizendo por quê (DEP-06); **Propor organismo (P-ORG-05)**: cerimônia que abre *decisão no comitê* (nunca botão que cria sala) e a proposta aparece na árvore; **Estatuto (P-ORG-06)**: cada regra com sua proveniência — não há painel de admin; **Busca (P-NAV-04)**: local e só da sala, com o porquê de não existir busca global.

Alternância **recém-chegada × veterana** na barra de topo demonstra a política anti-cegueira-a-avisos (§11): para quem está chegando, os avisos de honestidade aparecem por extenso e a checklist fica visível; para a veterana, encolhem. As strings de honestidade do protótipo são o rascunho que virou a **tabela única canônica** ([design system §11.1](../docs/07-paginas/00-design-system.md)).

## Decisões de protótipo (DS §12)

- **Dark-only** (pendência "tema claro sem valores" resolvida por compromisso explícito; defensável por opsec/D7).
- **`--ink-2` clareado** para `#8C97A6` (o valor original `#6B7684` reprovava o AA 4.5:1 medido na validação).
- **Identicon**: placeholder normativo (grade 5×5 espelhada, determinística do nome) até o esquema final.
- **Strings de honestidade**: as canônicas da **tabela única** ([DS §11.1](../docs/07-paginas/00-design-system.md)) — este arquivo foi o rascunho dela; divergência daqui em diante é defeito.
- Dados, nomes e organizações **fictícios**; nenhum dado real.

## Verificação

`node --check` no JS embutido (sintaxe) + capturas headless de Chromium das telas-chave (Panorama, Cartilha, Lição, Cota, Deliberação, Voto nominal no Congresso, Mural, Conta, Membros; no ciclo 4: Emendas, Minhas salas + árvore, Regime nos dois estados, Cota sob PIX identificado, e as cerimônias de destravar/recuperar/emparelhar) — conferindo render, isolamento de compartimento (Congresso mostra "Camarada Vértice"/fechadura 2, nunca o handle da Célula), coerência do censo e a ramificação por regime das folhas de metadados. No ciclo 5, os **comportamentos** foram exercitados ponta a ponta no headless (mensagem → fila offline → reconexão → enviada; proposta → decisão na lista → debate; emenda apresentada; convite gerado → lista; boas-vindas; organismo proposto → árvore; estatuto; busca) — zero erros de console.
