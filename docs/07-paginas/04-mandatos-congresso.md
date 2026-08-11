# 07/04 — Mandatos, eleições e congresso (MAN)

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-11 |
| **Depende de** | [07 — Páginas (índice)](README.md), [00-design-system](00-design-system.md), [02 — Domínio §2.2, §2.7, §6](../02-modelo-de-dominio.md), [03 — Cripto §7–8](../03-arquitetura-criptografica.md) |
| **Público** | designers e engenheiros ([técnico]) com seções [conceitual] |

> Aqui vive a única fonte de poder do sistema: o **mandato eleito, temporário e revogável** (I3, I4, I6). Toda tela desta área torna visível a **proveniência** e o **prazo** do poder (UX3) e o caminho de **prestação de contas** e **recall** (P4). O congresso é o organismo supremo e **temporário** (M13); credenciar seus delegados é a operação em **lote** que o MLS resolve com um só `Commit` (I9). O gabarito de 9 pontos ([README §5](README.md)) rege cada página.

---

## Da base ao congresso [conceitual]

```mermaid
sequenceDiagram
    participant Cel as Célula
    participant Del as Deliberação (eleição)
    participant Man as Mandato
    participant Cong as Congresso
    Cel->>Del: abre eleição de delegado (voto secreto)
    Del->>Man: apuração cria Mandato (titular/suplente)
    Man->>Cong: credenciamento (em lote — um Commit)
    Cong-->>Cel: atestado assinado ("delegado eleito da célula X")
    Note over Man,Cong: mandato expira ao fim do congresso; revogável por recall da célula
```

O poder **emana da base pela eleição** e **desce em decisões**; o mandato **volta à base** pela prestação de contas e pelo recall. Nenhum mandato fica fora do alcance de quem o criou — inclusive o do Comitê Central (P-MAN-06).

---

## P-MAN-01 — Eleição de delegado

1. **Objetivo.** Eleger delegado(s) por uma deliberação do tipo `eleicao` que **produz mandato** (I3).
2. **Quem chega.** Membros elegíveis da célula/organismo eleitor.
3. **Funcionalidades.** Abrir eleição (candidaturas na discussão); votar (tipicamente **secreto** — reusa P-DEL-06/07); apuração cria `Mandato` (titular/suplente/observador) com `periodo` e `revogavel`.
4. **Dinâmica.** É um subtipo de deliberação (máquina de estados de DEL); a diferença é o desfecho: em vez de resolução, **mandato** (doc 02 §2.7, fig. 4.3). Sem eleição válida registrada, ninguém figura como titular/suplente (I3).
5. **Experiência e layout.** Fluxo de deliberação com candidaturas; ao apurar, gera `MandateCard` com destino, prazo e revogabilidade.
6. **Estados.** Em discussão/candidatura; votação; mandato criado; sem quórum (arquivada).
7. **Restrições.** I3 (poder só por eleição), I5/I12 (voto), I4 (mandato temporário/revogável).
8. **Aberto.** Política de titular/suplente e substituição em ausência (doc 02 §7).

## P-MAN-02 — Meus mandatos

1. **Objetivo.** Ver e cumprir os mandatos que **eu** exerço.
2. **Quem chega.** Mandatários (delegados, dirigentes).
3. **Funcionalidades.** Listar meus mandatos (destino, prazo, estado); ver **pendências de prestação de contas** (P-MAN-04); alertas de expiração.
4. **Dinâmica.** O mandato **expira** ao fim do `periodo` (`estado→expirado`) e pode ser revogado antes (recall). A prestação de contas é **rotina**, não exceção (doc 01 A.5).
5. **Experiência e layout.** `MandateCard` por item, com a **fonte** (qual eleição/mandante) e o prazo em destaque; ação de "prestar contas".
6. **Estados.** Ativo, próximo de expirar, expirado, revogado; prestação de contas em atraso (destaque).
7. **Restrições.** I4; UX3 (proveniência e prazo à vista); P4 (prestação de contas).
8. **Aberto.** —

## P-MAN-03 — Mandatos do organismo

1. **Objetivo.** Ver quem representa o organismo, com prazo e caminho de recall.
2. **Quem chega.** Membros do organismo mandante.
3. **Funcionalidades.** Listar os mandatos que o organismo conferiu (quem, destino, prazo, estado); iniciar **recall** (P-MAN-05); ver relatórios de prestação de contas recebidos.
4. **Dinâmica.** Transparência do poder para quem o conferiu: a base vê seus delegados e pode **revogá-los** por deliberação (I4). O estado (`ativo`/`expirado`/`revogado`) e a `data_fim_efetiva` são explícitos.
5. **Experiência e layout.** Lista de `MandateCard`; ação "propor recall" leva a uma deliberação (nunca botão direto — UX4).
6. **Estados.** Com/sem mandatos ativos; recall em curso; mandato expirado.
7. **Restrições.** I4, I6 (poder é do mandante, não de quem opera o servidor).
8. **Aberto.** —

## P-MAN-04 — Prestação de contas

1. **Objetivo.** O mandatário reporta ao mandante — o fluxo ascendente do centralismo.
2. **Quem chega.** Mandatário (envia); mandante (recebe/aprecia).
3. **Funcionalidades.** Compor relatório vinculado ao mandato (é uma **Correspondência**, doc 02 §2.7/§2.4); anexar; enviar ao mandante; o mandante aprecia (pode desdobrar em deliberação).
4. **Dinâmica.** Prestação de contas periódica é **institucional** (doc 01 A.5); o relatório fica ligado ao mandato (`relatorios`). Reusa o compositor de correspondência (P-JOR-06) — e herda a dependência do envio cross-organismo [DEP-04] quando mandante e destino não compartilham grupo.
5. **Experiência e layout.** `CorrespondenceComposer` no contexto do mandato; histórico de relatórios no `MandateCard`.
6. **Estados.** Rascunho; enviado; apreciado; em atraso.
7. **Restrições.** P4; doc 02 §2.7 (relatórios vinculados).
8. **Aberto.** —

## P-MAN-05 — Recall / revogação

1. **Objetivo.** Revogar um mandato antes do prazo, por deliberação do mandante.
2. **Quem chega.** Membros do organismo mandante.
3. **Funcionalidades.** Abrir deliberação de recall; ao aprovar com quórum, `estado→revogado` com `data_fim_efetiva`, e — se o mandato dava acesso a organismos — a **remoção correspondente** (Commit `Remove`, PCS: perde o futuro).
4. **Dinâmica.** Recall é **deliberação do mandante** (I4, UX4) — não botão. A execução criptográfica (remoção/rekey) segue a decisão. Interage com disciplina (P-DEL-10) quando o recall acompanha sanção.
5. **Experiência e layout.** Fluxo de deliberação; `ConfirmDestructive` só na execução pós-quórum; o `MandateCard` passa a "revogado" com a resolução de origem.
6. **Estados.** Em deliberação; aprovado → executando remoção; rejeitado.
7. **Restrições.** I4, UX4; doc 03 §7 (Remove/PCS).
8. **Aberto.** —

## P-MAN-06 — Convocação de congresso

1. **Objetivo.** Convocar o congresso — ordinário (direção) ou **extraordinário** (a base, por limiar de células) — garantindo que **nenhum mandato escapa ao recall**.
2. **Quem chega.** Direção (ordinário); conjunto de células (extraordinário).
3. **Funcionalidades.** Convocação ordinária pela direção (publica no jornal, pauta e prazo — P-JOR-04); **convocação extraordinária** por um **limiar de células** (X% delibera a convocação), que habilita inclusive **revogar mandatos do Comitê Central**.
4. **Dinâmica.** Resolve o furo apontado na revisão (doc 02 §2.7): o `mandante` de um mandato do CC é o Congresso, que ao encerrar fica `dissolvido` e não poderia revogá-lo entre congressos. O estatuto define um **`mandante` persistente de recall** — a convocação de congresso extraordinário por limiar de células — de modo que o mandato mais poderoso **também** volta ao alcance da base. *Dependência sinalizada:* a **contagem do limiar entre células** é uma agregação cross-organismo (quem conta as deliberações de células que não compartilham grupo?) — mesmo mecanismo pendente da mensageria entre organismos [DEP-04].
5. **Experiência e layout.** Dois caminhos claros; o extraordinário mostra o progresso do **limiar** (quantas células já deliberaram a convocação), como um `QuorumMeter` distribuído entre células.
6. **Estados.** Convocação ordinária publicada; extraordinária acumulando células até o limiar; limiar atingido → congresso aberto.
7. **Restrições.** doc 02 §2.7 (recall de congresso); I4 (nenhum mandato fora de alcance); I6.
8. **Aberto.** ADR próprio do `mandante` persistente de recall (doc 02 §2.7 remete à fase seguinte); parâmetro X% no estatuto.

## P-MAN-07 — Painel do congresso

1. **Objetivo.** Conduzir o congresso: pauta, credenciamento, deliberações — como organismo **temporário** e supremo.
2. **Quem chega.** Delegados credenciados; observadores; a direção que o convocou.
3. **Funcionalidades.** Ver pauta e `periodo`; acessar o credenciamento (P-MAN-08); abrir/participar de deliberações do congresso (reusa DEL); ao encerrar, o congresso passa a `dissolvido` mas suas **resoluções persistem**.
4. **Dinâmica.** O congresso é um organismo (grupo MLS) **temporário** (M13); ao dissolver, o grupo se encerra, mas as resoluções vão para o arquivo durável (P-ORG-08) e **descem para toda a organização — resultado pretendido cuja base normativa está pendente [DEP-07]**: sob I13 literal, o congresso (filho da raiz, sem descendentes na árvore atual do doc 02) não vincularia ninguém; a emenda (árvore ORG→Congresso→CC, fiel ao doc 01 A.7, ou exceção tipada) é pré-condição de implementação. Elege a direção (mandatos).
5. **Experiência e layout.** Compartimento com cor própria e um selo de "temporário / período X–Y"; `DeliberationStepper` nas deliberações; contagem de delegados credenciados.
6. **Estados.** Em credenciamento; em sessão; encerrando; dissolvido (arquivo persiste).
7. **Restrições.** M13/doc 02 §2.2, §6; I13; I9 (composição em lote).
8. **Aberto.** Regras de quórum com titulares/suplentes ausentes (doc 02 §7).

## P-MAN-08 — Credenciamento de delegados

1. **Objetivo.** Credenciar em **lote** os delegados eleitos, provando "este delegado foi eleito pela célula X" **sem expor os demais membros** da célula. *(Honestidade [DEP-05]: o atestado atual usa o **pseudônimo global** — um rótulo estável que cruza célula→congresso→frente justamente para os alvos prioritários de repressão; a forma final do identificador no atestado — handle de destino com correlação custodiada pelo mandante — é matéria do ADR de identidade.)*
2. **Quem chega.** Delegados eleitos; a mesa/secretaria do congresso.
3. **Funcionalidades.** Cada mandato gera um **atestado assinado**; a admissão ao grupo MLS do congresso é **um `Commit` em lote** para centenas de delegados (I9), não uma rotação por delegado.
4. **Dinâmica.** O atestado prova a elegibilidade do delegado sem revelar a composição da célula de origem (doc 02 §6, passo 3). O lote elimina o custo O(n²) e a corrida do modelo anterior (I9, doc 03 §7.2).
5. **Experiência e layout.** Lista de credenciamento com seleção em lote; barra de progresso do `Commit` único; `DelegateAttestation` por delegado (também reutilizado na federação, FED).
6. **Estados.** Aguardando credenciamento; lote em processamento; credenciado; recusado.
7. **Restrições.** I9 (lote); doc 02 §6; A3 (não expor a célula de origem).
8. **Aberto.** —

## P-MAN-09 — Votação nominal em congresso

1. **Objetivo.** Registrar quem votou o quê, quando a tradição do congresso o exige.
2. **Quem chega.** Delegados credenciados.
3. **Funcionalidades.** Votação **aberta** (commit-reveal, reusa P-DEL-05) com registro nominal; útil para posições que devem ficar em ata.
4. **Dinâmica.** É o modo `aberto`: garante simultaneidade e deixa **registro nominal** (doc 03 §8.1). Coexiste com o voto secreto para as eleições internas do congresso (ex.: eleição da direção).
5. **Experiência e layout.** `OpenBallot`; `HonestyCallout`: *"Voto nominal — fica registrado quem votou o quê."*
6. **Estados.** Commit; reveal; apuração; não-revelado = abstenção.
7. **Restrições.** doc 03 §8.1; I5/I12.
8. **Aberto.** —

## Decisões em aberto da área

- **ADR do mandante persistente de recall** (P-MAN-06): formaliza o recall do CC via congresso extraordinário; parâmetro X% ([revisao-critica-2.md](../revisao-critica-2.md)).
- **[DEP-07] Congresso × I13** (P-MAN-07): emenda de árvore/invariante — pré-condição da descida das resoluções.
- **[DEP-04] Agregações cross-organismo** (P-MAN-04/06): prestação de contas e contagem do limiar extraordinário.
- **[DEP-05] Identificador nos atestados** (P-MAN-08): pseudônimo global vs. handle de destino.
- **Quórum de congresso com suplência/ausência** (P-MAN-07): política de substituição (doc 02 §7); fluxo "assumir suplência" sem página até o ADR.
- **Titular/suplente na eleição** (P-MAN-01): regra de promoção do suplente.
- **Pauta pré-congresso**: construção da pauta pela base (janela de teses/propostas das células → consolidação) — hoje a pauta só aparece pronta em P-MAN-06/07; compõe com Tendência/Plataforma (doc 01 C.1) e está registrada em revisao-critica-2.

## Referências

- [doc 02 §2.2, §2.7, §6](../02-modelo-de-dominio.md); [doc 03 §7–8](../03-arquitetura-criptografica.md).
- [Design system](00-design-system.md) — `MandateCard`, `DeliberationStepper`, `QuorumMeter`, `DelegateAttestation`, `CorrespondenceComposer`.
