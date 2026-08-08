# ADR-0006 — Voto secreto por assinatura cega + urna de chave dividida

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [03 §8](../03-arquitetura-criptografica.md), [02 §2.5–2.6](../02-modelo-de-dominio.md) (I5) |

## Contexto

Eleições e deliberações precisam de dois regimes distintos: votação **nominal aberta** (registro de quem votou o quê, tradição de congresso) e **voto secreto** (delegados, cargos), garantindo elegibilidade, unicidade (I5), sigilo e verificabilidade — sem prometer o que não se pode cumprir (resistência a coação).

## Decisão

- **Nominal aberta:** *commit-reveal* (`BLAKE2b(voto || sal)` assinado; revelação após prazo). Garante simultaneidade.
- **Secreta:** **assinatura cega (RFC 9474)** emitida pela comissão eleitoral, que valida elegibilidade **sem ver o voto**; a cédula é depositada cifrada, com a credencial não-rastreável, numa **urna cuja chave é dividida entre ≥2 escrutinadores (Shamir)**, aberta só em quórum. A deposição é feita por **canal anônimo (Tor)** — requisito, não opcional.

## Alternativas consideradas

- **Voto "secreto" só cifrado para a mesa** — rejeitado: a mesa (ou o servidor) liga voto→pessoa; não é sigilo real.
- **Apuração homomórfica + ZK (Helios/Belenios)** — adiada como evolução: dá verificabilidade ponta a ponta sem escrutinadores confiáveis, ao custo de complexidade; adotar esquema publicado quando entrar no escopo, sem reinventar.
- **Mixnet** — adiada: forte, porém pesada para o MVP.

## Consequências

- (+) Elegibilidade e conteúdo do voto ficam criptograficamente separados; nenhum escrutinador isolado abre a urna.
- (−) **Depende de canal anônimo na deposição**; sem Tor, o sigilo cai por correlação de IP/horário (declarado no doc 03 §8.2 e doc 06).
- (−) Não resiste a **coação/venda de voto** (o eleitor pode provar seu voto). Limite declarado, não escondido.
- (−) Introduz RSABSSA e Shamir na base de código — primitivas fora de libsodium, exigindo biblioteca auditada (decisão em aberto no doc 03 §10).
