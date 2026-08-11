# Protótipo de telas — ciclos 1–3

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

Alternância **recém-chegada × veterana** na barra de topo demonstra a política anti-cegueira-a-avisos (§11): para quem está chegando, os avisos de honestidade aparecem por extenso e a checklist fica visível; para a veterana, encolhem.

## Decisões de protótipo (DS §12)

- **Dark-only** (pendência "tema claro sem valores" resolvida por compromisso explícito; defensável por opsec/D7).
- **`--ink-2` clareado** para `#8C97A6` (o valor original `#6B7684` reprovava o AA 4.5:1 medido na validação).
- **Identicon**: placeholder normativo (grade 5×5 espelhada, determinística do nome) até o esquema final.
- **Strings de honestidade**: as canônicas das specs, verbatim — este arquivo serve de rascunho da "tabela única" pendente no DS §12.
- Dados, nomes e organizações **fictícios**; nenhum dado real.

## Verificação

`node --check` no JS embutido (sintaxe) + capturas headless de Chromium das telas-chave (Panorama, Cartilha, Lição, Cota, Deliberação, Voto nominal no Congresso, Mural, Conta, Membros) — conferindo render, isolamento de compartimento (Congresso mostra "Camarada Vértice"/fechadura 2, nunca o handle da Célula) e coerência do censo.
