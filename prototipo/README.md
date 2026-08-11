# Protótipo de telas — ciclo 1

**Um único arquivo autocontido** ([index.html](index.html)) — abra no navegador, sem servidor, sem dependência externa (coerente com D7: zero CDN, fontes de sistema, ícones inline). É uma **bancada de demonstração**, não o produto: dados 100% fictícios, e os selos tracejados `DEP-nn` marcam desenho em aberto (registro canônico em [docs/07-paginas/README.md §7](../docs/07-paginas/README.md)).

## O que demonstra

Telas do plano de prototipagem ([doc 07 §8](../docs/07-paginas/README.md)): Panorama (regra de conteúdo dos cartões), Mural (TrustChip → MetadataDisclosure, resolução descida, need-to-know), Seletor com **transição de espinha**, Deliberações (stepper, censo congelado na abertura — I12), Discussão com tendências, **cerimônia de voto secreto** (coleta k-de-n, janela de mistura, estado re-entrante — e o **fail-closed ao vivo**: derrube o Tor na bancada durante o depósito), Apuração com bulletin board em 3 números, Resolução (ata serifada, `substitui`, assinatura do organismo), Membros & papéis (proveniência, sem botão de remover), Emissão de convites (log intra-compartimento, chip condicionado), Cotização + voucher, Painel de Exposição contextual (com o infiltrado A2), e o onboarding F1 completo (convite → pseudônimo → **senha de uso diário → chave-mestra de papel** → Tor → pendente de admissão).

## Decisões de protótipo (1ª semana — DS §12)

- **Dark-only** no ciclo 1 (pendência "tema claro sem valores" resolvida por compromisso explícito; defensável por opsec/D7).
- **`--ink-2` clareado** para `#8C97A6` (o valor original `#6B7684` reprovava o AA 4.5:1 medido na validação).
- **Identicon**: placeholder normativo (grade 5×5 espelhada, determinística do nome) até o esquema final.
- **Strings de honestidade**: as canônicas das specs, verbatim — este arquivo serve de rascunho da "tabela única" pendente no DS §12.
- Dados, nomes e organizações **fictícios**; nenhum dado real.
