# 02 — Modelo de domínio

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-08 |
| **Depende de** | [00 — Visão](00-visao.md), [01 — Fundamentos](01-fundamentos-leninistas.md) |
| **Alimenta** | [03](03-arquitetura-criptografica.md), [04](04-federacao.md), [05](05-financiamento.md), [06](06-modelo-de-ameacas.md) |
| **Público** | todos ([conceitual]) + engenheiros ([técnico]) |

> Este documento define **as entidades do sistema, suas relações e as regras (invariantes) que nunca podem ser violadas**. As entidades de *organização* têm origem na tabela de mapeamento M1–M13 do [doc 01](01-fundamentos-leninistas.md), indicada entre colchetes; os mapeamentos M14 (finanças) e M15 (criptografia/opsec) alimentam os docs [05](05-financiamento.md) e [03](03-arquitetura-criptografica.md), não entidades daqui. O [doc 03](03-arquitetura-criptografica.md) introduz seu próprio vocabulário técnico (envelope, urna, escrutinador, credencial anônima) definido lá.

## 1. Visão geral [conceitual]

O sistema modela **uma organização como uma árvore de organismos**. Na base estão as células (grupos de militantes); acima, comitês eleitos; no topo, o congresso e a direção. Pessoas são **militantes** (pertencem a uma célula) ou **simpatizantes** (só acompanham o jornal público). Organismos tomam decisões (**deliberações**) que produzem **resoluções**; elegem **mandatos** (delegados); e se comunicam pelo **jornal**. Tudo isso vive dentro de uma **organização**, que é um servidor.

```mermaid
flowchart TD
    ORG["Organização<br/>(servidor)"]
    CC["Comitê Central / Direção"]
    CONG["Congresso<br/>(periódico)"]
    CL1["Comitê Regional Sul"]
    CL2["Comitê Regional Norte"]
    COM["Comissão de Finanças"]
    CA["Célula A<br/>(fábrica)"]
    CB["Célula B<br/>(bairro)"]
    CD["Célula C<br/>(setor)"]
    FR["Fração no Sindicato X"]

    ORG --> CC
    ORG --> CONG
    CC --> CL1
    CC --> CL2
    CC --> COM
    CL1 --> CA
    CL1 --> CB
    CL2 --> CD
    CA -.-> FR
    CB -.-> FR

    CA -. "elege delegados" .-> CONG
    CB -. "elege delegados" .-> CONG
    CD -. "elege delegados" .-> CONG
    CONG -. "elege" .-> CC
```

Linha cheia = relação estrutural (pai/filho na árvore). Linha tracejada = relações de eleição e de participação transversal (a fração reúne militantes de várias células).

## 2. Entidades [técnico]

Cada entidade é descrita por **descrição**, **atributos**, **relações** e **regras**. Atributos criptográficos (chaves, assinaturas) são apenas nomeados aqui; sua mecânica está no [doc 03](03-arquitetura-criptografica.md).

### 2.1 Usuário / Militante [M1, M3, M6]

**Descrição.** Uma pessoa participante, representada exclusivamente por chaves e um pseudônimo. **Não existe, em nenhum lugar do modelo, campo para nome real, e-mail, telefone ou qualquer PII** — a ausência é estrutural (P2), não uma configuração.

**Atributos.**

| Atributo | Descrição |
|---|---|
| `id` | Identificador auto-certificante derivado da chave pública de identidade (ver doc 03). |
| `pseudonimo` | Nome fictício escolhido pelo usuário; único dentro do servidor; mutável. |
| `chave_pub_identidade` | Chave pública de assinatura (Ed25519). Prova quem ele é. |
| `chave_pub_cifra` | Chave pública de cifração (X25519), certificada pela de identidade. Recebe conteúdo cifrado. |
| `nivel` | `simpatizante` ou `militante`. |
| `estado` | `ativo`, `afastado`, `desligado`. |

**Relações.** Um militante é membro de exatamente uma **célula-base** (I1) e pode participar adicionalmente de comissões e frações; pode deter mandatos; é membro de organismos que lhe dão acesso às respectivas chaves de época.

**Regras.** O servidor guarda apenas `pseudonimo`, chaves públicas, `nivel` e `estado`, além dos vínculos de participação. Autenticação é sempre por assinatura (doc 03).

**Simpatizante — caminho de acesso (esclarecido).** As publicações de escopo `publico` são legíveis **sem conta** (texto assinado, I7) — logo o simpatizante, no MVP, é um **leitor externo anônimo**, não um usuário com chaves. Só há um caminho de registro: o convite assinado por secretário de célula (doc 03 §4), que é o do **militante**. Se um dia se quiser um simpatizante *identificado como tal* (para receber conteúdo dirigido), define-se um registro próprio mais fraco (auto-registro sem admissão); por ora, `nivel = simpatizante` como *usuário* é reservado a esse caso futuro, e o leitor comum das publicações públicas não precisa de conta.

### 2.2 Organismo (abstrato) [M5, M7]

**Descrição.** Qualquer instância organizada. É a abstração central do modelo: célula, comitê, comissão, fração, congresso e direção são todos **subtipos** de organismo e compartilham a mesma espinha (árvore, membros com papéis, grupo MLS com seu epoch, deliberações).

**Atributos comuns.**

| Atributo | Descrição |
|---|---|
| `id` | Identificador do organismo. |
| `tipo` | `celula`, `comite`, `comissao`, `fracao`, `congresso`, `direcao`. |
| `nome` | Rótulo interno (não é PII de pessoa). |
| `pai` | Organismo superior na árvore (nulo apenas para a raiz lógica, a Organização). |
| `membros` | Conjunto de `(usuario, papel)`. |
| `epoch` | Inteiro; epoch corrente do grupo MLS do organismo — avança a cada mudança de composição ([doc 03 §7](03-arquitetura-criptografica.md), [ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md)). |
| `estado` | `ativo`, `suspenso`, `dissolvido`. |
| `estatuto_local` | Parâmetros herdados/sobrepostos do estatuto da organização (quórum, regra de maioria). |

**Papéis** (`papel` em `membros`): `secretario`, `tesoureiro`, `agitprop`, `membro`, e para organismos dirigentes `titular`/`suplente` (o mandato — 2.7). Papéis são **atribuídos e revogados por deliberação** com quórum, não por privilégio de sistema (Parte C.3 do doc 01; I11) — inclusive o `tesoureiro`, cuja revogabilidade é o que limita o risco de desvio financeiro (doc 05; doc 06 A8). O conjunto de papéis de direção de um organismo é o que a tradição chama de **buro** (não é um subtipo de organismo — ver glossário).

#### Subtipos de organismo

- **Célula [M1]** — organismo-base. Referência de tamanho: 3–15 membros. Único subtipo ao qual um militante pertence obrigatória e unicamente (I1). Delibera, elege delegados, coleta cotização, mantém correspondência com a redação. Tem `subtipo_celula`: `trabalho`, `territorio`, `setor` (Parte C.4 do doc 01).
- **Comitê [M7]** — organismo dirigente de um escopo (local, regional). Seus membros são **mandatos** eleitos pelas instâncias inferiores.
- **Comissão** — organismo funcional permanente (finanças, ética, agitprop/redação). Membros designados por deliberação do organismo que a cria.
- **Fração [M11]** — organismo transversal que reúne militantes de várias células que atuam numa organização externa. Não é célula-base de ninguém (I1 preservada). Para respeitar a árvore (I2), seu `pai` é o **comitê que a coordena** (ex.: o comitê responsável pela atuação sindical); as células de origem de seus membros são registradas como relação à parte (`fracao_alimentada_por`), não como múltiplos `pai`.
- **Congresso [M13]** — organismo deliberativo **temporário** e supremo. Tem `pauta`, `periodo` (abertura/encerramento) e um processo de **credenciamento** de delegados. Ao encerrar, passa a `dissolvido` mas suas resoluções persistem.
- **Direção / Comitê Central [M7]** — executivo eleito pelo congresso; dirige entre congressos. Mantém sub-organismos executivos (buro/secretariado) como comissões.

### 2.3 Organização / Servidor [M7]

**Descrição.** A entidade política que roda um servidor; **raiz** da árvore de organismos. Uma organização = um servidor (P6).

**Atributos.**

| Atributo | Descrição |
|---|---|
| `id` | Identificador da organização. |
| `nome` | Nome público da organização. |
| `chave_pub_organizacao` | Chave pública da organização; sua identidade na federação (doc 04). |
| `estatuto` | Parâmetros globais (ver abaixo). |

**Estatuto (parâmetros).** Quórum por tipo de deliberação; regra de maioria (simples/qualificada) por tipo; duração e teto de mandatos; proporção de delegados por número de membros; política de convites (quem pode emitir código de convite); tamanho mínimo/máximo de célula.

**Regras.** A organização não é um "superusuário": ela é o contexto e o conjunto de parâmetros. Nenhuma pessoa detém poder pelo simples fato de operar o servidor — poder é sempre mandato (I6).

### 2.4 Jornal, Publicação e Correspondência [M2, M10]

**Descrição.** O aparato de comunicação. O **Jornal** não é uma entidade própria: é uma *visão* das `Publicacao` agrupadas por `orgao_editor` e `escopo` — o "jornal central" é a visão das publicações da redação da organização; um "boletim de organismo" é a visão das publicações daquele organismo; o "jornal da frente" (doc 04) é a visão das publicações do organismo conjunto. Uma **Publicação** é uma edição/matéria; uma **Correspondência** é um informe que sobe da base para a redação.

**Publicação — atributos.**

| Atributo | Descrição |
|---|---|
| `id`, `titulo`, `corpo` | Conteúdo (cifrado, exceto escopo público — ver regra abaixo). |
| `orgao_editor` | Organismo (redação/comissão de agitprop) que assina a publicação. |
| `estado_editorial` | `rascunho` → `aprovada` → `publicada` → `arquivada`. |
| `escopo_circulacao` | `publico`, `interno_organizacao`, `interno_organismo`. |
| `assinatura` | Assinatura do órgão editor (doc 03). |

**Correspondência — atributos.** `remetente_organismo`, `destino` (redação ou instância superior), `corpo` (cifrado), `assinatura`. Modela o fluxo ascendente do centralismo democrático (M10).

**Regras.** Publicação de escopo `publico` é a **única exceção formal** à regra "tudo cifrado" (I7): ela é assinada mas legível, para alcançar simpatizantes e o exterior (ver ADR-0003). Escopos internos são cifrados para o organismo destinatário.

### 2.5 Deliberação [M4]

**Descrição.** O processo decisório de um organismo — a transformação do centralismo democrático em máquina de estados.

**Atributos.** `organismo`, `tipo` (`consultiva`, `deliberativa`, `eleicao`), `proposta`, `emendas`, `quorum`, `regra_maioria`, `modo_voto` (`aberto` ou `secreto`), `prazos` (por fase), `estado` (ver ciclo de vida na §4).

**Regras.** Toda deliberação passa obrigatoriamente por uma fase de **discussão** antes da **votação** (P4). O `modo_voto` determina a mecânica criptográfica (doc 03 §8): `aberto` usa commit-reveal (voto nominal simultâneo); `secreto` usa assinatura cega + urna. Uma eleição (`tipo = eleicao`) produz **mandatos** (2.7); uma deliberativa produz **resolução** (2.6).

### 2.6 Voto e Resolução [M4]

**Voto.** `deliberacao`, `eleitor` (ou credencial anônima, no voto secreto), `conteudo` (cifrado/às cegas conforme o modo), `assinatura`. Regra: **um voto por membro por deliberação** (I5), garantido por elegibilidade e unicidade (doc 03 §8).

**Resolução.** Decisão registrada de uma deliberação bem-sucedida: `deliberacao`, `texto`, `orgao` (que a emite), `escopo_vinculacao` (quais organismos obriga), `substitui` (id de resolução anterior que esta corrige/revoga — nullable), `assinatura_do_organismo`, `timestamp`. A resolução é uma **ata assinada** e imutável; correções são **novas** resoluções que apontam para a anterior via `substitui` (encadeamento auditável — I10). Propaga-se para os feeds dos organismos subordinados dentro do `escopo_vinculacao` — é o "de cima para baixo" do centralismo (P4).

> **Mecanismo da propagação descendente (esclarecido após a revisão).** Como cada organismo é um compartimento com chave própria, o comitê emissor **não** tem a chave das células-filhas. A resolução vinculante é, portanto, **reembalada pelo emissor** para os membros dos organismos no `escopo_vinculacao` (pelo mecanismo de grupo do [doc 03](03-arquitetura-criptografica.md), custo O(membros-alvo) assumido conscientemente) — não basta "publicar com a chave do comitê", que seria ilegível embaixo. Ver [revisao-critica.md](revisao-critica.md) §2-A.

### 2.7 Mandato / Delegação [M9]

**Descrição.** Poder conferido por eleição — a única fonte de poder no sistema (I6).

**Atributos.**

| Atributo | Descrição |
|---|---|
| `mandante` | Organismo que elegeu (a base do poder). |
| `mandatario` | Usuário que recebe o mandato. |
| `destino` | Organismo onde o mandato é exercido (ex.: o congresso, o comitê). |
| `tipo` | `titular`, `suplente`, `observador`. |
| `periodo` | Início e fim (o mandato **expira**). |
| `revogavel` | Verdadeiro por padrão; recall por deliberação do mandante. |
| `estado` | `ativo`, `expirado`, `revogado` — registra o desfecho (antes ausente). |
| `data_fim_efetiva` | Quando o mandato terminou de fato (expiração ou revogação). |
| `relatorios` | Correspondências de prestação de contas vinculadas ao mandato. |

**Regras.** Um mandato só nasce de uma deliberação `eleicao` registrada (I3). Expira automaticamente ao fim do `periodo` (`estado→expirado`). Pode ser revogado antes por nova deliberação do `mandante` (recall → `estado→revogado`, com `data_fim_efetiva`). Sem eleição válida, não há como um usuário figurar como `titular`/`suplente` em organismo dirigente.

**Recall de mandato de congresso (recém-esclarecido).** O `mandante` de um mandato do Comitê Central é o **Congresso**, que ao encerrar fica `dissolvido` e não pode mais deliberar — logo o mandato mais poderoso ficaria sem quem o revogasse entre congressos (I4 falharia justamente para ele). Solução: o estatuto define um **`mandante` persistente de recall** para mandatos de congresso — por exemplo, a convocação de **congresso extraordinário** por um limiar de células (X% das células delibera a convocação), que então pode revogar. Nenhum mandato fica fora do alcance da base. (Ver [revisao-critica.md](revisao-critica.md) §3, achado 7; registrar em ADR próprio na fase seguinte.)

### 2.8 Frente [M12]

**Descrição.** Acordo entre organizações (portanto, entre servidores) para ação comum. Detalhada no [doc 04](04-federacao.md); no modelo local, é referenciada por organismos conjuntos (um comitê da frente, um jornal da frente) cujos membros incluem delegados credenciados de outra organização.

**Atributos (visão local).** `acordo` (documento assinado pelas chaves das duas organizações), `escopo`, `validade`, `organismos_conjuntos`, `delegados_credenciados_externos`.

## 3. Invariantes

Regras que **nenhuma operação** pode violar. Numeradas e citáveis (I#).

- **I1 — Célula-base única.** Todo militante pertence a exatamente uma célula-base. Participações adicionais só em comissões e frações (nunca em segunda célula-base).
- **I2 — Árvore de organismos.** Os organismos formam uma árvore com raiz na organização; não há ciclos; todo organismo (exceto a raiz) tem exatamente um `pai`.
- **I3 — Poder só por eleição.** Um mandato (`titular`/`suplente` em organismo dirigente) só existe se derivar de uma deliberação `eleicao` registrada e válida.
- **I4 — Mandato é temporário e revogável.** Todo mandato tem `periodo` finito e pode ser revogado por deliberação do `mandante`.
- **I5 — Um voto por membro por deliberação.** Garantido por elegibilidade + unicidade (doc 03 §8). No voto secreto, a unicidade é imposta por **emissão limiar** de credencial (nenhuma mesa isolada cunha credenciais) + **uso único** verificado pela urna + conferência contra o censo congelado (I12) — não por confiança numa mesa de parte única (ver [ADR-0006](decisoes/adr-0006-voto-secreto-assinatura-cega.md) atualizado).
- **I6 — Sem superusuário.** Nenhum poder decorre de operar o servidor ou de qualquer atributo pessoal; todo poder é mandato (I3). O operador do servidor é adversário no modelo de ameaças (doc 06, A5).
- **I7 — Nenhum *conteúdo* em claro no servidor, salvo publicação pública.** O corpo de todo objeto é envelope cifrado (doc 03); a única exceção de *conteúdo* é a `Publicacao` de `escopo = publico`, assinada e legível por desenho (ADR-0003). *(Correção de sobreafirmação: o cabeçalho de roteamento, os timestamps, o grafo de filiação e o **`papel`** de cada membro permanecem visíveis ao servidor — a ACL depende disso; ver [doc 06 A3](06-modelo-de-ameacas.md) e a mitigação por credenciais anônimas no [ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md).)*
- **I8 — Quórum para vincular.** Uma resolução só é vinculante se a deliberação que a produziu atingiu o `quorum` e a `regra_maioria` do estatuto.
- **I9 — Época/epoch acompanha a composição.** Entrada ou saída de membro avança a época do organismo e dispara nova distribuição de chave de grupo (doc 03 §7). *(Sob MLS — [ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md) — o avanço é um `Commit` autenticado; admissões/remoções podem ser feitas **em lote** — ex.: credenciamento de um congresso — em vez de uma época por membro, evitando o custo O(n²) e a corrida da versão anterior.)*
- **I10 — Resolução é imutável.** Publicada, uma resolução não se altera; correções são novas resoluções que a referenciam via `substitui` (encadeamento auditável — P5).
- **I11 — Sanção só por deliberação.** *(Novo — [ADR-0009](decisoes/adr-0009-disciplina-como-deliberacao.md).)* Qualquer transição de `estado` de membro que restrinja direitos (censura, afastamento, `desligado`) só é válida como consequência de uma **deliberação com quórum** do organismo competente. A exclusão criptográfica é *executada* pelo sistema, mas *decidida* pela deliberação — não há exclusão unipessoal (fecha o "superusuário oculto"; reforça I6). O desligamento implica remoção de todas as filiações e revogação de todos os mandatos do usuário, com avanço de época.
- **I12 — Censo eleitoral congelado.** *(Novo.)* Ao abrir uma deliberação, a composição elegível (o "caderno eleitoral") é **congelada** até a apuração — mudanças de membros no meio não alteram o denominador do quórum (I8) nem a contagem de credenciais do voto secreto (I5). O número de elegíveis é publicável para conferência (bulletin board).
- **I13 — Vinculação dentro da subárvore.** *(Novo.)* O `escopo_vinculacao` de uma resolução ⊆ descendentes do `orgao` que a emite — um organismo não vincula quem está fora de sua subárvore (o "de cima para baixo" respeita a árvore I2).

## 4. Ciclos de vida [técnico]

### 4.1 Deliberação

```mermaid
stateDiagram-v2
    [*] --> Proposta
    Proposta --> Discussao: abre discussão
    Discussao --> Emendas: emendas apresentadas
    Emendas --> Discussao: novo turno
    Discussao --> Votacao: encerra discussão (prazo/quórum)
    Votacao --> Apuracao: prazo de voto encerrado
    Apuracao --> Resolucao: quórum atingido (I8)
    Apuracao --> Arquivada: sem quórum
    Resolucao --> [*]
    Arquivada --> [*]
```

### 4.2 Publicação

```mermaid
stateDiagram-v2
    [*] --> Rascunho
    Rascunho --> Aprovada: redação aprova
    Aprovada --> Publicada: assinada pelo órgão editor
    Publicada --> Arquivada: substituída/encerrada
    Rascunho --> Descartada
    Arquivada --> [*]
    Descartada --> [*]
```

### 4.3 Eleição de delegado (célula → congresso)

```mermaid
sequenceDiagram
    participant Cel as Célula
    participant Del as Deliberação (eleição)
    participant Man as Mandato
    participant Cong as Congresso

    Cel->>Del: abre eleição de delegado ao congresso
    Del->>Del: discussão + candidaturas
    Del->>Del: votação (modo secreto: urna cifrada)
    Del->>Man: apuração cria Mandato (titular/suplente)
    Man->>Cong: credenciamento do delegado
    Cong-->>Cel: delegado credenciado (atestado assinado)
    Note over Man,Cong: mandato expira ao fim do congresso e é revogável por recall da célula
```

## 5. Diagrama de entidades [técnico]

```mermaid
erDiagram
    ORGANIZACAO ||--o{ ORGANISMO : contem
    ORGANISMO ||--o{ ORGANISMO : "pai-filho"
    ORGANISMO ||--o{ MEMBRO : tem
    USUARIO ||--o{ MEMBRO : "participa via"
    ORGANISMO ||--o{ DELIBERACAO : realiza
    DELIBERACAO ||--o{ VOTO : recebe
    DELIBERACAO ||--o| RESOLUCAO : produz
    DELIBERACAO ||--o{ MANDATO : "elege (se eleicao)"
    USUARIO ||--o{ MANDATO : exerce
    ORGANISMO ||--o{ PUBLICACAO : edita
    ORGANISMO ||--o{ CORRESPONDENCIA : envia
    ORGANIZACAO ||--o{ FRENTE : "participa de"

    USUARIO {
        id id
        string pseudonimo
        bytes chave_pub_identidade
        bytes chave_pub_cifra
        enum nivel
        enum estado
    }
    ORGANISMO {
        id id
        enum tipo
        id pai
        int epoch
        enum estado
    }
    MEMBRO {
        id usuario
        id organismo
        enum papel
    }
    DELIBERACAO {
        id id
        enum tipo
        enum modo_voto
        int quorum
        enum estado
    }
    MANDATO {
        id mandante
        id mandatario
        id destino
        enum tipo
        daterange periodo
        bool revogavel
    }
    RESOLUCAO {
        id id
        text texto
        id orgao
        set escopo_vinculacao
        bytes assinatura
    }
```

## 6. Cenário narrativo: um congresso em seis passos [conceitual]

Para amarrar as entidades ao vocabulário do [doc 01](01-fundamentos-leninistas.md), segue o percurso de uma decisão real.

1. **Convocação.** A **direção** (2.2) publica no **jornal** (2.4) a convocação do **congresso** (2.2/M13), com pauta e prazo — uma `Publicacao` de escopo interno à organização.
2. **Eleição na base.** Cada **célula** (2.1/M1) abre uma **deliberação** do tipo `eleicao` (2.5) para escolher seu **delegado**. A discussão acontece; a votação usa **voto secreto** (urna cifrada — doc 03 §8). A apuração cria um **mandato** (2.7/M9).
3. **Credenciamento.** Os delegados eleitos são **credenciados** no congresso (fig. 4.3): cada mandato gera um atestado assinado que prova "este pseudônimo é delegado eleito da célula X", sem expor os demais membros da célula.
4. **Deliberação no congresso.** O congresso abre **deliberações** sobre a pauta. Militantes agrupam posições e plataformas na fase de discussão (Parte C.1 do doc 01); segue a votação.
5. **Resolução.** Cada deliberação bem-sucedida (com **quórum** — I8) produz uma **resolução** (2.6): ata assinada pelo organismo, imutável (I10).
6. **Vinculação descendente.** As resoluções propagam-se, dentro de seu `escopo_vinculacao`, para os feeds de todos os organismos subordinados. A base agora executa a decisão que ajudou a formar — o ciclo do centralismo democrático (P4) se fecha.

## 7. Decisões em aberto

- Granularidade de **tarefas** atribuíveis dentro de um organismo (M10) — modelar como entidade própria ou como atributo de correspondência? (Adiado para a fase de implementação.)
- Regras de **quórum em congresso** com delegados titulares/suplentes ausentes — política de substituição.
- Se **frações** podem, elas próprias, deliberar de forma vinculante ou apenas coordenar. (Provável: apenas coordenar; confirmar com organizações reais na revisão.)

## Referências

- [doc 01 — Fundamentos leninistas](01-fundamentos-leninistas.md), tabela M1–M15.
- [doc 03 — Arquitetura criptográfica](03-arquitetura-criptografica.md) para a mecânica de chaves, envelopes e voto.
