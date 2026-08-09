# Revisão crítica do desenho (2026-08)

| | |
|---|---|
| **Status** | estável (registro de revisão) |
| **Data** | 2026-08-09 |
| **Método** | cinco revisões adversariais independentes (fundamentos leninistas; arquitetura criptográfica; privacidade/ameaças; modelo de domínio/federação; financiamento/viabilidade), consolidadas |
| **Escopo revisado** | docs 00–06, ADRs 0001–0007, glossário |

> Este documento registra a crítica ao desenho da fase inicial e as **decisões de rumo** tomadas a partir dela. Ele é um artefato de governança: cada tema aponta a severidade, a **convergência** entre revisores (quando dois ou mais chegaram ao mesmo furo por caminhos distintos — o sinal mais forte de que o achado é real) e a correção. As decisões de rumo estão na §7.

## 1. Veredito geral

O trabalho tem duas qualidades reais e raras: **rastreabilidade** (conceito histórico M# → princípio P# → invariante I# → ADR) e **honestidade técnica** no [doc 06](06-modelo-de-ameacas.md). Mas os cinco revisores convergem para três falhas estruturais:

1. **O TCB está fora do modelo de ameaças.** O sistema modela o servidor e a rede como adversários, mas **não** modela os três componentes onde a confiança de fato reside: o **cliente / cadeia de suprimento**, o **tesoureiro** e a **mesa eleitoral**. São exatamente os pontos por onde o adversário real entra sem tocar no servidor.
2. **A honestidade mora no doc técnico; a superpromessa, no doc que os militantes leem.** O [doc 06](06-modelo-de-ameacas.md) é sóbrio, mas o [doc 00](00-visao.md)/[doc 01](01-fundamentos-leninistas.md) prometem "servidor cego", "resolve a lista de filiação" e "voto secreto" — que são, respectivamente, contornáveis por cliente comprometido, falsos em tempo real (o grafo trafega em claro) e condicionais a um Tor que o público não sustenta.
3. **O "coração" do projeto — o centralismo democrático (P4) — não tinha implementação**, nem criptográfica (a resolução não consegue descer) nem disciplinar (nada obriga a "unidade de ação").

## 2. Achados críticos

### A — Propagação descendente de resolução é criptograficamente impossível
*Severidade: crítico. Convergência: domínio + fundamentos.*

Cada organismo é um compartimento com **chave de época própria**, detida só por seus membros ([doc 03 §7](03-arquitetura-criptografica.md)). Os delegados de um comitê **não** são membros das células-filhas e não têm a chave delas — logo a resolução do comitê, cifrada com a chave do comitê, é **ilegível** pelas células que ela deve vincular. O escopo `interno_organizacao` ([doc 02 §2.4](02-modelo-de-dominio.md)) não tem chave definida em lugar nenhum. Do lado político, mesmo que descesse não haveria efeito: a Parte C.2 do doc 01 diz que o software "não decide sanções", e nada acontece a quem ignora a resolução. **"Vinculante" (I8/I10) era palavra sem mecanismo.** É a feature-título da visão.

**Correção:** difusão descendente explícita — a resolução vinculante é reembalada pelo emissor para os membros dos organismos no `escopo_vinculacao` (via o mecanismo de grupo do doc 03), custo O(membros-alvo) assumido; e a força vinculante vira consequência de deliberação (§7, decisão de disciplina).

### B — A criptografia de grupo por época é insegura como especificada
*Severidade: crítico. Convergência: cripto (4 achados) + domínio.*

- **`crypto_box_seal` é anônimo e não-autenticado:** qualquer um — em especial o servidor — pode fabricar um pacote de chave de época e entregar **chaves diferentes a membros diferentes** (equivocação). Faltam assinatura do admissor e *confirmation tag* de grupo.
- **Sem post-compromise security:** como a `pk_enc` não muda na rotação, comprometer o `sk_enc` de um membro (adversário A4, previsto!) dá leitura **permanente** de todo o conteúdo futuro.
- **Envelope sem transcrição:** sem `msg_id`/contador/encadeamento, replay, reordenação e drop silencioso pelo servidor são indetectáveis. A "cadeia de hashes no jornal" que o doc 06 cita como mitigação **não existe** na spec.
- **Downgrade de época:** a época corrente não é autenticada ao remetente → cifra sob chave antiga que um removido ainda tem.

**Correção (decisão §7): adotar MLS (RFC 9420)** — árvore assinada, `tree_hash`, `confirmation_tag`, PCS e forward secrecy — como piso do MVP; envelope com transcrição em cadeia; avanço de época autenticado.

### C — O grafo de filiação (ativo nº 2) trafega em claro para o servidor adversário
*Severidade: crítico. Convergência: privacidade + cripto + domínio.*

`remetente` + `organismo_destino` em claro no envelope entregam o organograma pseudônimo **em tempo real** (não só em apreensão). Dois vetores não estavam sequer no doc 06: o **`papel`** de cada membro (a ACL exige que o servidor saiba quem é secretário/tesoureiro — os alvos prioritários) e o **grafo de recrutamento** (o convite assinado revela qual secretário apadrinhou quem). O doc 00 chama a lista de filiação de "primeiro alvo de qualquer repressão" — e o desenho a entrega ao adversário.

**Correção (decisão §7): credenciais de membro anônimas (BBS+/KVAC)** — o remetente prova "sou membro autorizado do organismo O para o tipo T" **sem revelar qual membro**; o servidor verifica ACL sem aprender identidade nem grafo por mensagem. Reconcilia ACL-no-servidor + ocultação de grafo + *deniability*.

### D — O voto secreto protege o sigilo, não a integridade
*Severidade: crítico. Convergência: cripto + domínio.*

A assimetria está invertida: o sigilo é protegido por limiar (Shamir entre escrutinadores), mas a **integridade** (emissão de credenciais) fica numa **mesa de parte única** que pode cunhar credenciais ilimitadas — indetectável, porque o vínculo credencial↔membro foi destruído por desenho. Falta prevenção de duplo-depósito. A cédula anônima também **viola** a regra "tudo é envelope" (ADR-0003 exige `remetente`+assinatura).

**Correção (decisão §7):** emissão **limiar** da assinatura cega (mesa distribuída k-de-n); **bulletin board** público (nº de credenciais emitidas vs. censo congelado); urna por **DKG + VSS** (ninguém detém a chave inteira) e decifração limiar; declarar a cédula anônima como tipo de objeto próprio, exceção formal ao envelope.

### E — Cliente/cadeia de suprimento e tesoureiro não são adversários
*Severidade: crítico. Convergência: privacidade + financiamento.*

Toda a confidencialidade repousa na premissa "o binário que o usuário roda = o código publicado", com build reprodutível rebaixado a "meta futura". Loja de apps pode ser compelida a um build direcionado → exfiltra a seed → E2E irrelevante. E o **voucher** concentra toda a confiança no tesoureiro (nunca modelado): só sobe o agregado autoassinado por ele, ninguém reconcilia os vouchers → o desvio é **indetectável por desenho** (menos auditável que um caderno de papel).

**Correção (Camada 0): adversários A7 (cliente/supply-chain), A8 (insider financeiro), A9 (mesa eleitoral)** no doc 06; build reprodutível + binary transparency como **requisito**; auditoria do voucher ancorada fora do tesoureiro (dupla assinatura + log append-only de vouchers).

### F — Financiamento: default legal invertido
*Severidade: crítico. Fonte: financiamento.*

O "modo anônimo" recomendado como padrão é estruturalmente o mecanismo de **caixa dois** para partido registrado. Faltam Lei 9.504/97, crime eleitoral (art. 350 CE), DME/Receita. **Correção (Camada 0):** default **transparente** quando a organização se declara partido registrado, com bloqueio ativo do modo anônimo nesse caso; ampliar o aviso legal.

## 3. Achados importantes

| # | Achado | Fonte | Tratamento |
|---|---|---|---|
| 1 | "Perda de chave = perda de conta" + Tor + fingerprint são irreais p/ leigos e induzem backup inseguro da seed | privacidade, finanças | Recuperação social → MVP (§7 futura); guia de backup |
| 2 | Tor "por padrão" sem **fail-closed** → desanonimização silenciosa | privacidade | doc 06/doc 03: exigir fail-closed |
| 3 | Push (FCM/APNs) fora do modelo → device token liga pseudônimo a conta real | privacidade | doc 06: adversário de push; polling/token desacoplado |
| 4 | Desafio de login **não é assinado pelo servidor** → relay | cripto | ADR-0002 atualizado: servidor assina com `chave_pub_organizacao` |
| 5 | Envelope Ed25519 = prova não-repudiável (passivo criminal); falta *deniability* | cripto | resolvido por credencial anônima (C); documentar tradeoff |
| 6 | Separação de domínio só no login; concatenações sem length-prefix | cripto | doc 03: rótulo de domínio em todo objeto assinado; COSE/CBOR |
| 7 | Mandato sem estado de revogação; Resolução sem `substitui`; **recall do CC impossível entre congressos** | domínio | Camada 0 (campos+ciclo); ADR sobre recall do CC |
| 8 | Quem muda `estado→desligado` = expulsão unipessoal de facto (superusuário oculto, fere I6) | fundamentos, domínio | Camada 0: I11 (sanção só por deliberação) |
| 9 | Somem quadro profissional, candidato/vetting, jornal como arma/escola, linha de massas | fundamentos | doc 01: acrescentar (fidelidade) |
| 10 | "Ferramenta neutra" insustentável; uso dual (org clandestina maligna) não modelado | fundamentos | doc 00/01: assumir que embute governança; doc 06: nota de uso dual |
| 11 | I9 O(n²) com corrida no credenciamento de congresso; §7.1 vs §7.2 contradizem qual chave o novo membro recebe | cripto, domínio | resolvido por MLS (add em lote, epoch autenticado) |
| 12 | Escopo MVP de federação (Nível 2 vinculante) depende de itens "em aberto" (completude do espelho, revogação) | domínio | rebaixar MVP p/ Nível 1 até prova de completude |
| 13 | Argon2id calibrado p/ servidor, não p/ seed em repouso sob ataque offline; falta entropia de senha e keystore de hardware | cripto | doc 03: tier SENSITIVE, diceware, hardware keystore |

## 4. Correções factuais (aplicadas na Camada 0)

- **§1 de 1903:** a fórmula de **Martov venceu** a votação; os rótulos bolchevique/menchevique vieram das eleições aos **órgãos centrais** após a saída do Bund e dos economistas — não do §1. (doc 01 A.3)
- **Teses de organização de 1921:** **criticadas pelo próprio Lênin** no IV Congresso (1922) como "russas demais". (A.5)
- **Frente única:** formulada pelo CEIC (dez/1921) e IV Congresso (1922), não pelo III; "marchar separados, golpear juntos" é máxima militar (Moltke). (A.10)
- **Datas:** *Carta a um camarada* publicada em 1903/04; a lista de 5 pontos do centralismo é codificação de 1906. (A.2/A.4)
- **Argumento de continuidade** pré/pós-1905 suavizado indevidamente (em 1904 Lênin defende o "burocratismo" contra o "democratismo"): o equilíbrio democracia/centralismo é **conjuntural** (legal vs. clandestino). (A.2/A.4)
- **GNU Taler** não usa "a mesma primitiva RFC 9474": usa assinaturas cegas chaumianas (RSA), migrando para Schnorr cego. (doc 05 §3)

## 5. Coerência interna apontada

- **P4 vs. C.2** (vinculante prometido, sanção negada) — resolvido pela decisão de disciplina (§7).
- **C.6 "neutro" vs. I1–I10** (invariantes que gravam um modelo de governo) — reconhecido no doc 00/01.
- **M5/M15 (doc 01) vs. A2/A3 (doc 06)** — o doc conceitual superafirmava a konspiratsiya; tom alinhado.
- **I7 "nada em claro"** era falso (cabeçalho, grafo, papéis) — reescrito para "nenhum **conteúdo** em claro".
- **"Jornal"** tratado como entidade sem existir no ER; **"buro"** com três sentidos divergentes — alinhados.

## 6. Higiene de repositório

`origin/main` é um commit-raiz vazio (base criada para viabilizar o PR num repositório vazio). **Não** se faz fast-forward de `main` enquanto o PR #1 (`claude/…` → `main`) está aberto — isso colapsaria o diff do PR. A higiene (main como tronco efetivo e branch default) **resolve-se ao mergear o PR #1**; depois disso, `main` conterá todo o conteúdo e a branch de trabalho pode ser aposentada.

## 7. Decisões de rumo tomadas

1. **Criptografia — reprojetar.** Adotar **MLS (RFC 9420)** como piso do MVP (substitui a chave de época por sealed box) e **credenciais de membro anônimas (BBS+/KVAC)** para ACL sem exposição de grafo. Registrado em **ADR-0008** (substitui ADR-0004); ADR-0002 e ADR-0006 atualizados. Reprojeto detalhado do [doc 03](03-arquitetura-criptografica.md).
2. **Centralismo — modelar disciplina.** Sanção (censura, afastamento, desligamento) e a força vinculante de resoluções passam a ser **consequência de uma deliberação com quórum** (espelhando I3). Registrado em **ADR-0009**; nova invariante I11; resolve o "superusuário oculto" da exclusão.
3. **Entrega.** Este documento de revisão + aplicação da **Camada 0** (correções factuais, campos/invariantes faltantes, adversários A7–A9, alinhamento de honestidade, correção de financiamento) na branch do PR #1.

## 8. O que permanece em aberto (próximas fases)

- Reprojeto detalhado completo do voto (bulletin board, DKG/VSS, cast-or-audit / verificabilidade E2E) e da federação (prova de completude do espelho, revogação de credencial).
- Recuperação social de identidade como parte do MVP.
- Tratamento de push notifications, fail-closed de rede e keystore de hardware na spec.
- Fidelidade leninista: quadro profissional, candidato/vetting, jornal como arma/escola, linha de massas — decidir o que entra no domínio vs. o que é escopo social.
