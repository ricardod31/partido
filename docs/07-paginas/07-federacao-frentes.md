# 07/07 — Federação e frentes (FED)

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-11 |
| **Depende de** | [07 — Páginas (índice)](README.md), [00-design-system](00-design-system.md), [04 — Federação](../04-federacao.md), [ADR-0005](../decisoes/adr-0005-federacao-por-acordo-bilateral.md), [06 — Ameaças A5](../06-modelo-de-ameacas.md) |
| **Público** | designers e engenheiros ([técnico]) com seções [conceitual] |

> A federação é a tática da **frente única** virada arquitetura: *"marchar separados, golpear juntos"* (doc 01 A.10). É **bilateral e explícita** — primeiro o acordo político entre as direções, depois o peering técnico (ADR-0005); nada além do que o acordo autoriza atravessa a fronteira. A propriedade inegociável: **uma organização nunca expõe sua base de membros à outra** (doc 04 §4). O gabarito de 9 pontos ([README §5](README.md)) rege cada página.

---

## Ciclo de vida de uma frente [conceitual]

```mermaid
sequenceDiagram
    participant A as Organização A
    participant B as Organização B
    A->>B: proposta de frente (escopo, prazo, nível)
    B->>A: contraproposta / aceite
    A->>A: delibera e assina com a chave da organização
    B->>B: delibera e assina com a chave da organização
    A->>B: troca de acordos coassinados + fingerprints (fora da banda)
    Note over A,B: habilita allowlist mútua; cria organismos conjuntos
    A-->>B: espelha jornal da frente e resoluções conjuntas
    Note over A,B: ao término, aposenta as chaves da frente; cada lado retém seu arquivo
```

---

## P-FED-01 — Frentes

1. **Objetivo.** Ver e navegar as frentes de que a organização participa.
2. **Quem chega.** Dirigentes (gestão); militantes (leitura do jornal da frente espelhado).
3. **Funcionalidades.** Listar frentes (escopo, nível, prazo, estado); entrar no painel de uma frente (P-FED-04); ver o estado do acordo.
4. **Dinâmica.** Cada frente é um acordo bilateral com organismos conjuntos. A base lê a frente **pelo próprio servidor** (espelho) — nunca se conecta ao servidor da outra organização (doc 04 §4).
5. **Experiência e layout.** Lista com nível (1–3) e prazo; frentes têm sua própria cor de compartimento.
6. **Estados.** Proposta/negociação; ativa; em dissolução; encerrada (arquivo retido).
7. **Restrições.** ADR-0005; doc 04 §4 (base não cruza a fronteira).
8. **Aberto.** Federação multilateral (frente de 3+ — doc 04 §8).

## P-FED-02 — Propor frente

1. **Objetivo.** Propor uma frente a outra organização, definindo escopo, prazo e nível.
2. **Quem chega.** Direção/mandato competente.
3. **Funcionalidades.** Definir escopo (o que a frente fará), validade, e **nível de interface** (1 jornal comum, 2 organismo conjunto, 3 processos conjuntos — **MVP: 1–2**); enviar à outra organização.
4. **Dinâmica.** Primeiro o acordo, depois o peering (ADR-0005). O nível 3 (voto conjunto entre bases distintas) é evolução (doc 04 §5) — a UI o mostra como indisponível no MVP, com o porquê.
5. **Experiência e layout.** Assistente de proposta; seletor de nível com descrição honesta de cada um; nível 3 desabilitado com nota.
6. **Estados.** Rascunho; enviada; contraproposta recebida.
7. **Restrições.** doc 04 §5 (níveis; MVP 1–2); ADR-0005.
8. **Aberto.** Nível 3 — elegibilidade e sigilo entre bases (doc 04 §8).

## P-FED-03 — Acordo da frente

1. **Objetivo.** Coassinar o acordo com a **chave da organização** e estabelecer a confiança na chave da outra parte **fora da banda**.
2. **Quem chega.** Mandato que custodia a chave da organização (ex.: sob controle do CC).
3. **Funcionalidades.** Deliberar internamente (doc 04 §3); **assinar o acordo com a `chave_pub_organizacao`**; **verificar o fingerprint** da chave da outra organização por canal seguro; habilitar a allowlist mútua.
4. **Dinâmica.** A confiança na chave é estabelecida **fora da banda** (encontro de delegados, canal já confiável — doc 04 §2), para impedir injeção de chave falsa (ataque de diretório, A5). Aqui a conferência de fingerprint **é** a defesa correta (é bootstrap de confiança entre organizações, não a "safety number" cotidiana que o doc 06 A5 desaconselha como defesa primária).
5. **Experiência e layout.** `AgreementCoSign` (`Ceremony`) + `FingerprintVerify` em mono (design system §5.4); deixa claro que a assinatura é **do organismo/organização**, não pessoal.
6. **Estados.** Aguardando deliberação interna; aguardando coassinatura da outra parte; fingerprint não conferido (bloqueia); ativa.
7. **Restrições.** doc 04 §2/§3; A5 (verificação fora da banda); dependência: assinatura da organização (chave da org — doc 04 §2).
8. **Aberto.** —

## P-FED-04 — Painel da frente

1. **Objetivo.** Operar o organismo conjunto: jornal da frente e comitê da frente.
2. **Quem chega.** Delegados credenciados dos dois lados; base (leitura do jornal espelhado).
3. **Funcionalidades.** Ler/publicar no **jornal da frente** (nível 1); deliberar no **comitê da frente** (nível 2, vinculante **apenas dentro do escopo da frente**); ver resoluções conjuntas.
4. **Dinâmica.** O comitê da frente é um organismo (grupo MLS) cujos membros são os **delegados credenciados** dos dois lados (doc 04 §4). Os objetos são os **mesmos envelopes** do doc 03 — a federação não introduz cripto nova. Escritas vão ao **servidor-sede**; o outro lado mantém **espelho somente-leitura** (doc 04 §6).
5. **Experiência e layout.** Compartimento da frente (cor própria); indica o **servidor-sede** e o estado do espelho; reusa DEL/JOR dentro do escopo da frente.
6. **Estados.** Espelho sincronizado/atrasado; sede indisponível (escrita bloqueada, leitura do espelho segue); nível 1 (só jornal, sem comitê).
7. **Restrições.** doc 04 §4/§5/§6; vinculação restrita ao escopo da frente.
8. **Aberto.** Protocolo de sincronização do espelho e prova de completude (doc 04 §8).

## P-FED-05 — Delegados credenciados externos

1. **Objetivo.** Credenciar quem representa cada lado na frente — **sem expor a base**.
2. **Quem chega.** Direção/secretaria de cada organização.
3. **Funcionalidades.** Emitir **atestado assinado** ("este pseudônimo é delegado credenciado da organização B para a frente F"); admitir os delegados externos ao organismo conjunto; revogar credenciais.
4. **Dinâmica.** Só delegados credenciados cruzam a fronteira (doc 04 §4): o servidor de A vê apenas os delegados que **B credenciou**, nunca o grafo de filiação de B. Reusa `DelegateAttestation` (também usado em MAN).
5. **Experiência e layout.** Lista de delegados externos com seus atestados; TrustChip: *"Você vê os delegados que a outra organização credenciou — não a base dela."*
6. **Estados.** Credenciado; revogado; atestado inválido.
7. **Restrições.** doc 04 §4 (não expor base); A3.
8. **Aberto.** Revogação de credencial em tempo hábil no servidor parceiro (doc 04 §8).

## P-FED-06 — Identidade federativa da organização

1. **Objetivo.** Publicar e manter a identidade federativa da organização.
2. **Quem chega.** Mandato que custodia a chave da organização.
3. **Funcionalidades.** Publicar `/.well-known/partido-org` (documento **assinado** pela chave da organização, doc 04 §2); **rotacionar** a chave (nova assinada pela anterior — continuidade verificável).
4. **Dinâmica.** A `chave_pub_organizacao` é a identidade na federação (doc 02 §2.3, doc 04 §2). A rotação preserva a cadeia de confiança (nova chave assinada pela antiga).
5. **Experiência e layout.** Painel com o documento well-known, o fingerprint em mono (para a conferência fora da banda de P-FED-03) e o histórico de rotação.
6. **Estados.** Publicado; rotação em curso (cadeia de continuidade); chave comprometida (procedimento de rotação de emergência).
7. **Restrições.** doc 04 §2; A5 (identidade federativa verificável).
8. **Aberto.** —

## Decisões em aberto da área

- **Nível 3** (processos decisórios conjuntos): elegibilidade e sigilo de voto entre bases (doc 04 §5/§8).
- **Sincronização do espelho** (P-FED-04): frequência, resolução de conflitos, prova de completude (doc 04 §8).
- **Federação multilateral** (P-FED-01): allowlists e organismos conjuntos de 3+ (doc 04 §8).
- **Revogação de credencial de delegado** no parceiro em tempo hábil (P-FED-05, doc 04 §8).

## Referências

- [doc 04](../04-federacao.md) (todo); [ADR-0005](../decisoes/adr-0005-federacao-por-acordo-bilateral.md); [doc 06 A5](../06-modelo-de-ameacas.md).
- [Design system](00-design-system.md) — `FrontCard`, `AgreementCoSign`, `FingerprintVerify`, `DelegateAttestation`.
