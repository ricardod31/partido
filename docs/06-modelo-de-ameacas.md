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

- O **cliente é livre e auditável**. A correspondência entre o binário que o usuário roda e o código publicado **não é premissa e sim requisito** a ser garantido (build reprodutível + binary transparency) — porque o cliente é a raiz de confiança e todo o TCB. Quando esse requisito falha, cai-se no adversário **A7**; esta é a fronteira mais importante do modelo.
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

**O que obtém.** O disco contém: chaves **públicas**, pseudônimos, envelopes **cifrados** (que ele não lê) e — o ponto crítico — **o grafo de organismos e membros em claro**, porque o servidor precisa dele para roteamento e ACL (doc 03 §6.2). Esse grafo é **pseudônimo**, mas **correlacionável**, e inclui dois subgrafos especialmente sensíveis que a versão anterior omitia: (i) o **`papel` de cada membro** — a ACL exige saber quem é secretário/tesoureiro de cada célula, que são os **alvos prioritários** de repressão; e (ii) o **grafo de recrutamento** — o convite de registro é assinado por um secretário específico, então o servidor sabe **quem apadrinhou quem, para dentro de qual organismo e quando** (uma árvore de proveniência que, subindo dos recrutas aos secretários, entrega a liderança celular).

**Mitigações.** Zero PII (não há nomes reais a apreender — ADR-0001); retenção mínima (timestamps truncados, expurgo de metadados antigos); **onion service** (o servidor pode estar em jurisdição/local menos acessível); conteúdo permanece cifrado e ilegível. **Mitigação estrutural em curso ([ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md)):** credenciais de membro anônimas (BBS+/KVAC) permitem ao servidor verificar ACL **sem** aprender identidade nem grafo por mensagem, e o convite pode ser cegado (o servidor valida que é um convite legítimo sem saber qual secretário o assinou). Como passo barato imediato, **handles por-organismo não-vinculáveis** (em vez de um `user_id` global que junta todos os organismos de uma pessoa) já quebram o *join* completo numa apreensão.

**Limite — declarado sem rodeios.** Enquanto as credenciais anônimas não estiverem implementadas, **o grafo de filiação pseudônimo (com papéis e proveniência de recrutamento) é o maior ativo exposto numa apreensão** — e o servidor o reconstrói **em tempo real**, não só sob apreensão (ver A5). Chamar "ACL sem o servidor conhecer o grafo" de mera "pesquisa em aberto" subdimensiona: há caminho deployável (credencial anônima, sharding de organismos entre servidores, identificadores rotativos) — é decisão de roadmap (ADR-0008), não impossibilidade.

### A4 — Comprometimento do dispositivo do militante

**Capacidade.** Acesso ao aparelho de um membro — apreensão desbloqueada, malware, coação.

**O que obtém.** A **chave privada** daquele membro e o **histórico local** dos organismos dele. Com a chave, pode se passar por ele até a revogação.

**Mitigações.** Seed cifrada em repouso com Argon2id (doc 03 §3.2); bloqueio de aplicativo; opção de **não reter histórico local** ou retê-lo por prazo curto; rotação de época ao detectar comprometimento (revoga o membro, corta acesso futuro — I9). Sub-chaves por dispositivo revogáveis (futuro) limitariam o dano a um aparelho.

**Amplificação sem *post-compromise security*.** No desenho original (chave de época por sealed box), comprometer o `sk_enc` de um membro dava leitura **permanente** de todo o conteúdo futuro (os novos pacotes de época continuavam selados para a mesma chave). A adoção de **MLS** ([ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md)) corrige isso: o *ratchet* dá *post-compromise security* — após a remoção/atualização do membro comprometido, o adversário perde o acesso ao futuro.

**Coação legal / entrega compelida de chave.** Em jurisdições com *key-disclosure laws*, o usuário pode ser **compelido a entregar a passphrase/seed**; um secretário compelido a entregar sua chave permite cunhar convites e (sem MLS) forjar até a revogação. Mitigações a considerar (fase futura): passphrase de coação / negação plausível e **sub-chaves por dispositivo revogáveis** para que uma chave compelida seja escopada e revogável.

**Limite.** Um dispositivo **desbloqueado nas mãos do adversário** entrega o que o usuário vê. Criptografia protege dados em repouso e em trânsito, não a tela de um aparelho aberto sob coação.

### A5 — Servidor malicioso ou coagido (sem apreensão total)

**Capacidade.** O operador do servidor (ou quem o coage) age contra os usuários enquanto opera normalmente.

**O que obtém / pode fazer.** **Não lê** conteúdo E2E. Mas **pode**: negar serviço, **reter ou reordenar** mensagens, **minerar metadados** (grafo, horários, volumes) e — o ataque mais sério — **tentar substituir chaves públicas** na distribuição da chave de grupo (doc 03 §7.1) ou no diretório, para se inserir num organismo (**ataque de diretório / MitM de chaves**).

**Mitigações.** A defesa robusta vem do **MLS** ([ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md)): `tree_hash` + `confirmation_tag` fazem todos os membros confirmarem a **mesma** visão de chave e composição, o que impede a **equivocação** (entregar chaves diferentes a membros diferentes) — o ataque que a distribuição por sealed box não detectava. O envelope passa a ter **transcrição em cadeia** (`msg_id` + contador + hash do anterior), tornando replay, reordenação e drop **detectáveis** pelos clientes — a "cadeia de hashes" que antes só existia na prosa deste documento agora é especificada no [doc 03 §6](03-arquitetura-criptografica.md). **Key transparency** (registro público verificável de chaves) permanece como evolução. A verificação de *fingerprint* fora da banda é defesa **secundária**, não a principal — ver limite.

**Limite.** O servidor sempre pode **negar serviço** e **observar metadados**. **Não se deve contar com a conferência humana de fingerprint como defesa primária** (a taxa real de conferência de "safety number" em sistemas como o Signal é próxima de zero); por isso a defesa central é criptográfica (MLS + transcrição), não comportamental.

### A6 — Sybil e flooding

**Capacidade.** Criar muitas identidades falsas para infiltrar, manipular votações ou inundar o sistema.

**O que obtém.** Sem defesa: peso desproporcional em deliberações, ruído, sobrecarga.

**Mitigações.** **Registro por convite assinado** pelo secretário da célula (doc 03 §4): não se cria identidade válida sem um convite emitido por um organismo. Isso ancora a criação de contas na estrutura real e encarece o Sybil.

**Limite.** Um secretário malicioso (ou coagido) pode emitir convites falsos — recai em A2 (infiltração) no nível daquele organismo.

### A7 — Cliente comprometido e cadeia de suprimento *(novo — TCB)*

**Capacidade.** O adversário não ataca o servidor: ataca **o software que o usuário roda**. Loja de aplicativos legalmente compelida a entregar um *build* direcionado a um alvo; mantenedor coagido a assinar um release malicioso; servidor de atualização comprometido; dependência envenenada.

**O que obtém.** **Tudo.** O cliente é a raiz de confiança e todo o TCB (doc 03 §1); um cliente adulterado exfiltra a `seed` ou o plaintext **antes** da cifragem — o E2E e o "servidor cego" tornam-se irrelevantes, sem o servidor fazer nada. Para celular, a loja é o caminho de entrega padrão.

**Mitigações.** **Build reprodutível** com *rebuilders* independentes e **binary transparency** (log público de binários) — tratados como **requisito**, não "meta futura"; distribuição estilo F-Droid com verificação de reprodutibilidade; auto-verificação de assinatura no cliente; minimizar superfície de atualização automática. **Limite:** contra um alvo específico com adversário estatal e loja cooperante, a garantia é frágil sem que o próprio usuário verifique o *build* — o que quase ninguém faz. Este é o **maior buraco** do modelo original, que não tinha este adversário.

### A8 — Insider financeiro (tesoureiro desviante ou coagido) *(novo)*

**Capacidade.** O tesoureiro de uma célula coleta cotização em espécie e emite vouchers (doc 05).

**O que obtém / pode fazer.** No desenho ingênuo, **desviar caixa de forma indetectável**: como só sobe o agregado autoassinado por ele e ninguém reconcilia os vouchers, ele pode subdeclarar (embolsar a diferença) ou inflar (emitir para amigos).

**Mitigações.** Auditoria ancorada **fora** do tesoureiro (doc 05 §4): agregado **coassinado pelo secretário** + **log append-only de vouchers emitidos** (série por época), permitindo conferir "nº de vouchers ↔ agregado" sem revelar pagadores; papel de tesoureiro **eleito e revogável** por deliberação (I11). **Limite:** colusão tesoureiro+secretário volta a esconder o desvio; a defesa final é a rotação e a prestação de contas conferível.

### A9 — Mesa/comissão eleitoral maliciosa *(novo)*

**Capacidade.** A comissão que emite as credenciais cegas de voto (doc 03 §8).

**O que obtém / pode fazer.** Como as credenciais são **não-rastreáveis** por desenho, uma mesa de parte única pode **cunhar credenciais extras** e encher a urna (*ballot stuffing*) de forma **indetectável** — a integridade do voto, não o sigilo, é o ponto fraco.

**Mitigações ([ADR-0006](decisoes/adr-0006-voto-secreto-assinatura-cega.md) atualizado).** **Emissão limiar** da assinatura cega (mesa distribuída k-de-n — nenhuma parte isolada cunha); **bulletin board** público com o nº de credenciais emitidas conferível contra o **censo congelado** (I12); **uso único** de credencial imposto pela urna (contra duplo-depósito). **Limite:** verificabilidade E2E (o eleitor conferir que seu voto foi contado) só com esquema tipo Helios/Belenios — futuro; o MVP é honestamente **não verificável ponta a ponta**.

### Adversários combinados *(nota)*

Os adversários acima **não agem isolados** no cenário real de repressão. Os combos mais perigosos: **A5 (servidor) + A2 (infiltrado)** — o infiltrado dá o plaintext e a identidade civil da sua célula, o servidor dá o grafo completo + timing; juntos desanonimizam **além** das células do infiltrado, porque a identificação propaga pelo grafo. **A3 + A2** idem, sob apreensão. A propriedade de "raio de dano limitado" da compartimentação (A2) **degrada abruptamente** quando o grafo do servidor está disponível ao mesmo adversário — mais uma razão para as credenciais anônimas de A3.

### Nota de uso dual

Este modelo trata adversários **contra** a organização. É preciso registrar o inverso: a mesma infraestrutura (pseudônima, servidor-cego, compartimentada, de entrada por convite, com filiação difícil de enumerar) é substrato **ótimo para qualquer organização clandestina de comando** — inclusive criminosa, insurrecional violenta ou seita. Não há mitigação técnica para isso sem quebrar as próprias propriedades que protegem uma organização legítima; é um **limite ético declarado** do projeto (ver [doc 01 C.6](01-fundamentos-leninistas.md)), não um problema resolvido.

## 5. Tabela "protege / não protege" [conceitual]

A seção mais importante do documento. **Leia antes de confiar sua segurança ao sistema.**

| O sistema **protege** | O sistema **NÃO protege** |
|---|---|
| Conteúdo de mensagens, atas e jornal interno contra o servidor e contra quem apreende o servidor (E2E) | Contra um **infiltrado** que é membro legítimo (ele lê o que seu organismo lê) |
| Identidade civil dos membros (zero PII no servidor) | O **grafo de filiação pseudônimo** (com papéis e proveniência de recrutamento), visível ao servidor **em tempo real** — não só sob apreensão — até as credenciais anônimas (ADR-0008) entrarem |
| Conteúdo em trânsito contra MitM (assinaturas + MLS) | **Metadados de participação** (quem fala com qual organismo, quando) — e, **sem Tor**, o IP; a degradação precisa ser **fail-closed** (recusar em vez de cair para clearnet em silêncio) |
| Sigilo do voto secreto, **se** depositado por canal anônimo | Contra **coação/venda de voto**; e a **integridade** do voto depende de emissão limiar + bulletin board (mesa de parte única cunha credenciais — A9) |
| Acesso futuro de quem sai de um organismo (PCS via MLS) | Um **dispositivo desbloqueado** nas mãos do adversário; e a **entrega compelida de chave** por lei |
| Contra criação em massa de contas (registro por convite) | Contra **análise de tráfego global** ou **negação de serviço** pelo servidor |
| — | Contra um **cliente/app comprometido ou build direcionado** (A7): contorna todo o E2E — depende de build reprodutível |
| — | Contra o **tesoureiro que desvia** (A8) e a **mesa eleitoral que frauda** (A9), sem as âncoras de auditoria/limiar |
| — | Contra o **provedor de push** (FCM/APNs): device token liga o pseudônimo a uma conta real e vaza o timing das mensagens |

## 6. Rastreabilidade: ameaça → mitigação → onde está especificada [técnico]

| Ameaça | Mitigação | Especificação |
|---|---|---|
| A1 MitM de conteúdo | Assinatura de envelope + MLS + TLS | doc 03 §6 |
| A1 IP / padrões | Tor / onion service **fail-closed**, padding | doc 03 §9 |
| A2 infiltração | Compartimentação por organismo | doc 03 §7; doc 01 M5 |
| A3 apreensão — conteúdo | E2E, zero PII, retenção mínima | doc 03 §6; ADR-0001 |
| A3 apreensão — grafo/papel/recrutamento | **Credenciais anônimas (BBS+/KVAC)**; handles por-organismo — em implantação | ADR-0008; doc 03 §9 |
| A4 dispositivo | Argon2id (tier SENSITIVE) + keystore de hardware; PCS via MLS | doc 03 §3.2; ADR-0008 |
| A4 coação legal | Passphrase de coação, sub-chaves revogáveis — **futuro** | doc 03 §10 |
| A5 equivocação de chaves | **MLS** (tree_hash + confirmation_tag); transcrição em cadeia | ADR-0008; doc 03 §6-7 |
| A5 metadados | Retenção mínima; credenciais anônimas; sealed sender | ADR-0008; doc 03 §9 |
| A6 Sybil | Registro por convite assinado (cegado) | doc 03 §4 |
| A7 cliente/supply-chain | **Build reprodutível + binary transparency (requisito)** | doc 03 §1; §9 (em aberto) |
| A8 insider financeiro | Coassinatura secretário + log de vouchers; papel revogável | doc 05 §4-5; doc 02 I11 |
| A9 mesa eleitoral | Emissão limiar + bulletin board + censo congelado | ADR-0006; doc 02 I5/I12 |
| Voto — sigilo | Assinatura cega + urna por DKG/VSS + Tor + mistura | doc 03 §8; ADR-0006 |
| Push (FCM/APNs) | Polling sobre Tor / token desacoplado — **em aberto** | doc 03 §9 (em aberto) |

As lacunas (linhas marcadas "em implantação", "futuro" ou "em aberto") estão registradas como **decisões em aberto** nos documentos de origem — não são esquecimentos.

## 7. Apêndice: contraprova STRIDE [técnico]

| Categoria STRIDE | Coberto por |
|---|---|
| **S**poofing (falsidade de identidade) | Autenticação por assinatura de nonce (doc 03 §5); `user_id` auto-certificante (ADR-0001) |
| **T**ampering (adulteração) | AEAD com cabeçalho como AD + assinatura de envelope (doc 03 §6); resoluções imutáveis (I10) |
| **R**epudiation (repúdio) | Transcrição em cadeia + MLS (A5). **Contrapartida declarada:** assinar cada envelope com identidade nominal é prova **não-repudiável** de autoria — um passivo criminal numa apreensão. Por isso a autoria ao servidor migra para **credencial anônima** (ADR-0008), que preserva *deniability*; o não-repúdio deixa de ser tratado como puro ganho |
| **I**nformation disclosure (vazamento) | E2E via MLS (doc 03 §7); zero PII (ADR-0001). **Limite forte:** grafo/papéis/recrutamento em claro (A3) até as credenciais anônimas; push (FCM/APNs) fora do E2E |
| **D**enial of service | Reconhecido como **não mitigado** contra servidor malicioso (A5); federação/espelho (doc 04 §6) dá alguma resiliência |
| **E**levation of privilege | Sem superusuário (I6); poder só por mandato eleito (I3); ACL por membro no envelope (doc 03 §6.2) |

## 8. Princípios de segurança operacional que o software incentiva [conceitual]

O software não substitui a opsec (P8), mas empurra na direção certa. **Importante:** um incentivo comportamental **não é um controle de segurança** — a experiência (PGP, Signal) mostra que disciplina que depende do usuário quase não acontece. Por isso, onde possível, a propriedade é **imposta** pelo sistema, não sugerida:

1. **Pseudônimo desde o início** — o onboarding orienta a escolher um pseudônimo **sem relação** com a identidade real. *(Limite: um dispositivo → pseudônimos correlacionáveis por rede/timing; não insinuar não-vinculabilidade que a camada de transporte desfaz.)*
2. **Tor por padrão, fail-closed** — o cliente prioriza o onion service e, **sem canal anônimo confirmado, recusa** operações sensíveis (depositar voto, sobretudo) com aviso não-dispensável, em vez de cair para clearnet em silêncio. Embarcar *bridges*/pluggable transports para redes que bloqueiam Tor.
3. **Células pequenas** — limite de tamanho **imponível pelo servidor** (não só sugerido no estatuto), limitando o raio de dano da infiltração (A2).
4. **Need-to-know como padrão de visibilidade** — novo membro não recebe o histórico por padrão (doc 03 §7).
5. **Defesa de chaves é criptográfica, não humana** — a integridade de chave/composição vem do MLS (confirmation_tag), não da conferência de fingerprint; esta é secundária e **não é creditada** como a mitigação principal de A5.
6. **Recuperação de conta** — "perda de chave = perda de conta" induz backup inseguro da seed (foto, nuvem), recriando o PII removido pelo ADR-0001. Por isso a **recuperação social** (re-atestação por quórum da célula) é trazida para perto do MVP, com guia de backup seguro da frase mnemônica.
7. **Educação embutida** — mensagens curtas explicam o que o sistema **não** protege (esta §5). É apoio à decisão, **não** um controle: nada de segurança depende só dela.

## 9. Decisões em aberto

- **Credenciais de membro anônimas (A3, ADR-0008)** — ACL no servidor sem exposição de grafo/papel; caminho de implementação (BBS+/KVAC), revogação e custo.
- **Cliente / cadeia de suprimento (A7)** — build reprodutível, *rebuilders*, binary transparency, canal de distribuição verificável; tratar como **requisito**.
- **Push notifications** — evitar FCM/APNs ou desacoplar o token da identidade; ou polling sobre Tor (custo de bateria/tráfego).
- **Fail-closed de rede** — especificar o comportamento exato do cliente sem Tor disponível.
- **Coação legal / entrega compelida (A4)** — passphrase de coação, negação plausível, sub-chaves revogáveis.
- **Recuperação social de identidade** — protocolo de re-atestação pela célula, para o MVP.
- **Verificabilidade E2E do voto (A9)** — Helios/Belenios / cast-or-audit, como evolução do MVP não-verificável.
- **Key transparency (A5)** — registro verificável de chaves.

## Referências

- [doc 03 — Arquitetura criptográfica](03-arquitetura-criptografica.md); [doc 04 — Federação](04-federacao.md); [doc 05 — Financiamento](05-financiamento.md).
- Metodologia: análise por adversário + STRIDE (Microsoft) como contraprova.
