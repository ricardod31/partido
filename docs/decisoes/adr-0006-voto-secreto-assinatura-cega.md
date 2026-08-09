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

**Atualização (2026-08-09) — integridade não pode ser de parte única.** A revisão crítica ([revisao-critica.md](../revisao-critica.md) §2-D) mostrou uma assimetria invertida: o sigilo é protegido por limiar (Shamir), mas a **integridade** (quantas credenciais existem) ficava numa **mesa de parte única** que pode cunhar credenciais ilimitadas de forma indetectável, e não havia prevenção de duplo-depósito. Correções incorporadas:
- **Emissão limiar** da assinatura cega (mesa distribuída k-de-n) — nenhuma parte isolada cunha credencial;
- **Quadro público (bulletin board)** com o número de credenciais emitidas, conferível contra o **censo eleitoral congelado** (nova invariante I12) — sobre-emissão vira detectável;
- **Urna por DKG + VSS** (geração distribuída de chave, shares verificáveis) e **decifração limiar** — a chave nunca é reconstruída num único ponto (Shamir puro pressupõe um *dealer* que conhece a chave inteira);
- **Uso único de credencial** imposto pela urna (nova invariante I-voto), prevenindo duplo-depósito;
- **Mistura/lote com atraso** entre emissão e deposição, contra correlação temporal emissão→deposição mesmo sobre Tor.
A cédula anônima é declarada **tipo de objeto próprio**, exceção formal à regra "tudo é envelope" (ver [ADR-0003](adr-0003-servidor-rejeita-nao-envelope.md), atualizado). Verificabilidade E2E (Helios/Belenios / cast-or-audit de Benaloh) permanece como evolução — o MVP é honestamente rotulado como **não verificável ponta a ponta**.

## Alternativas consideradas

- **Voto "secreto" só cifrado para a mesa** — rejeitado: a mesa (ou o servidor) liga voto→pessoa; não é sigilo real.
- **Apuração homomórfica + ZK (Helios/Belenios)** — adiada como evolução: dá verificabilidade ponta a ponta sem escrutinadores confiáveis, ao custo de complexidade; adotar esquema publicado quando entrar no escopo, sem reinventar.
- **Mixnet** — adiada: forte, porém pesada para o MVP.

## Consequências

- (+) Elegibilidade e conteúdo do voto ficam criptograficamente separados; nenhum escrutinador isolado abre a urna.
- (−) **Depende de canal anônimo na deposição**; sem Tor, o sigilo cai por correlação de IP/horário (declarado no doc 03 §8.2 e doc 06).
- (−) Não resiste a **coação/venda de voto** (o eleitor pode provar seu voto). Limite declarado, não escondido.
- (−) Introduz RSABSSA e Shamir na base de código — primitivas fora de libsodium, exigindo biblioteca auditada (decisão em aberto no doc 03 §10).
