# 06 — Modelo de ameaças

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-08 |
| **Depende de** | [03 — Arquitetura criptográfica](03-arquitetura-criptografica.md), [04 — Federação](04-federacao.md), [05 — Financiamento](05-financiamento.md) |
| **Público** | todos ([conceitual]) |

> Este documento diz, **sem eufemismo, contra o que o sistema protege e contra o que ele não protege**. Um projeto que promete segurança e entrega uma parte dela é mais perigoso do que um que é claro sobre seus limites — porque as pessoas tomam risco com base na promessa. A honestidade aqui é um requisito de segurança (P8).

## 1. Método [conceitual]

Analisamos por **adversário**: descrevemos quem quer atacar a organização, o que ele consegue fazer, o que obtém e o que o desenho faz (ou não faz) contra ele. É mais legível do que uma lista abstrata de vulnerabilidades. Ao final, um **apêndice STRIDE** (§7) serve de contraprova de cobertura para engenheiros.

## 2. Ativos a proteger [conceitual]

Em ordem aproximada de sensibilidade:

1. **Identidades** — as chaves privadas dos membros e a ligação (fora do sistema) entre pseudônimo e pessoa real.
2. **O grafo de filiação** — quem pertence a qual organismo. É o ativo que a repressão histórica sempre buscou primeiro; no sistema, é o mais difícil de esconder por completo (§5, A3).
3. **Conteúdo das deliberações e do jornal interno** — o que a organização discute e decide.
4. **Votos** — o sigilo do voto secreto.
5. **Registros financeiros** — a ligação entre pessoa e contribuição.
6. **Disponibilidade** — a capacidade de a organização continuar operando.
7. **A própria existência da associação** — em alguns contextos, o simples fato de existir e quem a compõe.

## 3. Premissas [técnico]

- O **cliente é livre e auditável**; assume-se que o binário que o usuário roda corresponde ao código publicado (reprodutibilidade de build é meta de implementação).
- As **primitivas criptográficas** do [doc 03](03-arquitetura-criptografica.md) são sólidas se bem implementadas (libsodium).
- O **operador do servidor pode ser adversário** (I6): o desenho não confia no servidor para confidencialidade.
- A **segurança operacional dos membros é imperfeita** (P8): pessoas erram, reusam dispositivos, são pressionadas.

## 4. Adversários [conceitual]

### A1 — Vigilância de rede (passiva e ativa)

**Capacidade.** Observa o tráfego entre clientes e servidor (provedor, Estado com acesso à rede); na forma ativa, tenta *man-in-the-middle* (MitM).

**O que obtém.** Sem anonimização de rede: **endereços IP** e **padrões de tráfego** (quem se conecta ao servidor da organização, quando, com que volume) — ou seja, a **existência** e o ritmo da atividade, mesmo sem ler conteúdo.

**Mitigações.** TLS no transporte + a assinatura dos envelopes (doc 03 §6) impedem MitM de **conteúdo**. **Tor / onion service** (doc 03 §9) removem o IP e dificultam o mapeamento de quem fala com o servidor. Padding em buckets reduz o vazamento por tamanho.

**Limite.** Sem Tor, A1 vê que "este IP fala com o servidor da organização" — um metadado forte. Análise de tráfego global (um adversário que vê a rede inteira) não é derrotada por este projeto.

### A2 — Infiltrado (membro legítimo malicioso)

**Capacidade.** É um membro de verdade, com chave válida, dentro de um ou mais organismos.

**O que obtém.** **Tudo o que seus organismos veem** — a criptografia E2E **não ajuda em nada** contra ele, porque ele é um destinatário legítimo.

**Mitigações.** A única defesa é estrutural: **compartimentação** (M5, need-to-know). Como cada organismo é um compartimento criptográfico próprio (doc 03 §7), o infiltrado vaza **apenas o que suas células veem**, não a organização inteira. Células pequenas e necessidade real de acesso limitam o dano.

**Limite.** É um problema **social**, não técnico. O software reduz o raio de dano; não impede a infiltração. Nenhuma criptografia resolve isto.

### A3 — Apreensão do servidor

**Capacidade.** O adversário toma posse física ou lógica do servidor (busca e apreensão, invasão, coação do provedor).

**O que obtém.** O disco contém: chaves **públicas**, pseudônimos, envelopes **cifrados** (que ele não lê) e — o ponto crítico — **o grafo de organismos e membros em claro**, porque o servidor precisa dele para roteamento e ACL (doc 03 §6.2). Esse grafo é **pseudônimo**, mas **correlacionável**: mostra a estrutura da organização e quem (por pseudônimo) está onde.

**Mitigações.** Zero PII (não há nomes reais a apreender — ADR-0001); retenção mínima (timestamps truncados, expurgo de metadados antigos); **onion service** (o servidor pode estar em jurisdição/local menos acessível); conteúdo permanece cifrado e ilegível.

**Limite — declarado sem rodeios.** **O grafo de filiação pseudônimo é o maior ativo exposto numa apreensão.** O desenho o minimiza (pseudônimos, sem PII, retenção curta) mas **não o elimina**, porque roteamento e controle de acesso exigem saber quem é membro de quê. Reduzir isso ainda mais (por exemplo, ACL sem o servidor conhecer o grafo) é problema de pesquisa em aberto (§6 e doc 03 §10).

### A4 — Comprometimento do dispositivo do militante

**Capacidade.** Acesso ao aparelho de um membro — apreensão desbloqueada, malware, coação.

**O que obtém.** A **chave privada** daquele membro e o **histórico local** dos organismos dele. Com a chave, pode se passar por ele até a revogação.

**Mitigações.** Seed cifrada em repouso com Argon2id (doc 03 §3.2); bloqueio de aplicativo; opção de **não reter histórico local** ou retê-lo por prazo curto; rotação de época ao detectar comprometimento (revoga o membro, corta acesso futuro — I9). Sub-chaves por dispositivo revogáveis (futuro) limitariam o dano a um aparelho.

**Limite.** Um dispositivo **desbloqueado nas mãos do adversário** entrega o que o usuário vê. Criptografia protege dados em repouso e em trânsito, não a tela de um aparelho aberto sob coação.

### A5 — Servidor malicioso ou coagido (sem apreensão total)

**Capacidade.** O operador do servidor (ou quem o coage) age contra os usuários enquanto opera normalmente.

**O que obtém / pode fazer.** **Não lê** conteúdo E2E. Mas **pode**: negar serviço, **reter ou reordenar** mensagens, **minerar metadados** (grafo, horários, volumes) e — o ataque mais sério — **tentar substituir chaves públicas** na distribuição da chave de grupo (doc 03 §7.1) ou no diretório, para se inserir num organismo (**ataque de diretório / MitM de chaves**).

**Mitigações.** Contra a troca de chaves: **verificação de *fingerprint* fora da banda** entre membros (e entre organizações, doc 04 §2), **alerta de mudança de chave** no cliente (TOFU — confiar-na-primeira-vez e avisar em qualquer mudança), e um **log append-only assinado** — a cadeia de hashes das publicações e atas no **jornal** (P5) funciona como registro verificável pelos clientes: o servidor não consegue reescrever o passado sem que a cadeia quebre. **Key transparency** (registro público verificável de chaves) é a evolução robusta (futuro).

**Limite.** O servidor sempre pode **negar serviço** e **observar metadados**. A detecção de troca de chaves depende de os usuários **realmente conferirem** fingerprints / prestarem atenção aos alertas (P8).

### A6 — Sybil e flooding

**Capacidade.** Criar muitas identidades falsas para infiltrar, manipular votações ou inundar o sistema.

**O que obtém.** Sem defesa: peso desproporcional em deliberações, ruído, sobrecarga.

**Mitigações.** **Registro por convite assinado** pelo secretário da célula (doc 03 §4): não se cria identidade válida sem um convite emitido por um organismo. Isso ancora a criação de contas na estrutura real e encarece o Sybil.

**Limite.** Um secretário malicioso (ou coagido) pode emitir convites falsos — recai em A2 (infiltração) no nível daquele organismo.

## 5. Tabela "protege / não protege" [conceitual]

A seção mais importante do documento. **Leia antes de confiar sua segurança ao sistema.**

| O sistema **protege** | O sistema **NÃO protege** |
|---|---|
| Conteúdo de mensagens, atas e jornal interno contra o servidor e contra quem apreende o servidor (E2E) | Contra um **infiltrado** que é membro legítimo (ele lê o que seu organismo lê) |
| Identidade civil dos membros (zero PII no servidor) | O **grafo de filiação pseudônimo**, visível ao servidor e a quem o apreende |
| Conteúdo em trânsito contra MitM (assinaturas + TLS) | **Metadados de participação** (quem fala com qual organismo, quando) sem Tor |
| Sigilo do voto secreto (assinatura cega + urna dividida), **se** depositado por canal anônimo | Contra **coação/venda de voto** (o eleitor pode provar como votou) |
| Acesso futuro de quem sai de um organismo (rotação de época) | Um **dispositivo desbloqueado** nas mãos do adversário |
| Contra criação em massa de contas (registro por convite) | Contra **análise de tráfego global** ou **negação de serviço** pelo servidor |

## 6. Rastreabilidade: ameaça → mitigação → onde está especificada [técnico]

| Ameaça | Mitigação | Especificação |
|---|---|---|
| A1 MitM de conteúdo | Assinatura de envelope + TLS | doc 03 §6 |
| A1 IP / padrões | Tor / onion service, padding | doc 03 §9 |
| A2 infiltração | Compartimentação por organismo | doc 03 §7; doc 01 M5 |
| A3 apreensão — conteúdo | E2E, zero PII, retenção mínima | doc 03 §6; ADR-0001 |
| A3 apreensão — grafo | Minimização (pseudônimos, retenção) — **parcial** | doc 03 §9; §8 abaixo (em aberto) |
| A4 dispositivo | Argon2id em repouso, revogação por época | doc 03 §3.2, §7.2 |
| A5 troca de chaves | Fingerprint fora da banda, alerta, log assinado no jornal | doc 03 §7; doc 04 §2 |
| A5 metadados | Retenção mínima, sealed sender (futuro) | doc 03 §9 |
| A6 Sybil | Registro por convite assinado | doc 03 §4 |
| Voto — elegibilidade/sigilo | Assinatura cega + urna dividida + Tor | doc 03 §8; ADR-0006 |
| Financiamento — pessoa↔pagamento | Voucher offline via tesoureiro; só agregados no servidor | doc 05 §4-5 |

As lacunas (linhas marcadas "parcial" ou "futuro") estão registradas como **decisões em aberto** nos documentos de origem — não são esquecimentos.

## 7. Apêndice: contraprova STRIDE [técnico]

| Categoria STRIDE | Coberto por |
|---|---|
| **S**poofing (falsidade de identidade) | Autenticação por assinatura de nonce (doc 03 §5); `user_id` auto-certificante (ADR-0001) |
| **T**ampering (adulteração) | AEAD com cabeçalho como AD + assinatura de envelope (doc 03 §6); resoluções imutáveis (I10) |
| **R**epudiation (repúdio) | Tudo assinado (Ed25519); log append-only no jornal (A5) |
| **I**nformation disclosure (vazamento) | E2E por época (doc 03 §7); zero PII (ADR-0001); minimização de metadados (§9) — **com os limites A3/A5** |
| **D**enial of service | Reconhecido como **não mitigado** contra servidor malicioso (A5); federação/espelho (doc 04 §6) dá alguma resiliência |
| **E**levation of privilege | Sem superusuário (I6); poder só por mandato eleito (I3); ACL por membro no envelope (doc 03 §6.2) |

## 8. Princípios de segurança operacional que o software incentiva [conceitual]

O software não substitui a opsec (P8), mas empurra na direção certa:

1. **Pseudônimo desde o início** — o onboarding orienta a escolher um pseudônimo **sem relação** com a identidade real e a nunca revelá-la no sistema.
2. **Tor por padrão** — o cliente prioriza a conexão via onion service.
3. **Células pequenas** — o estatuto sugere limites de tamanho que limitam o raio de dano da infiltração (A2).
4. **Need-to-know como padrão de visibilidade** — novo membro não recebe o histórico por padrão (doc 03 §7.2).
5. **Conferência de fingerprint** — o cliente pede e facilita a verificação fora da banda, sobretudo ao entrar em organismo novo ou ao ver alerta de mudança de chave (A5).
6. **Educação embutida** — mensagens curtas no fluxo explicam o que o sistema **não** protege (esta §5), no momento em que a decisão importa.

## 9. Decisões em aberto

- **Reduzir a exposição do grafo de filiação (A3)** ao servidor — ACL sem o servidor conhecer a composição plena dos organismos; pesquisa em aberto.
- **Sealed sender (A5/metadados)** — esconder o remetente do servidor sem perder ACL.
- **Key transparency (A5)** — registro verificável de chaves como evolução do log do jornal.
- **Reprodutibilidade de build do cliente** (premissa §3) — como o usuário verifica que roda o código auditado.

## Referências

- [doc 03 — Arquitetura criptográfica](03-arquitetura-criptografica.md); [doc 04 — Federação](04-federacao.md); [doc 05 — Financiamento](05-financiamento.md).
- Metodologia: análise por adversário + STRIDE (Microsoft) como contraprova.
