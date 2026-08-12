# ADR-0005 — Federação por acordo bilateral explícito (allowlist), não aberta

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [04 — Federação](../04-federacao.md) |

## Contexto

Organizações precisam cooperar em frentes comuns (M12) sem se fundir nem expor suas bases. É preciso escolher o modelo de federação entre servidores: aberto (qualquer servidor fala com qualquer outro) ou fechado (só por acordo).

## Decisão

Federação **bilateral e explícita**: dois servidores só trocam dados após as organizações firmarem um **acordo coassinado** pelas respectivas chaves de organização e habilitarem uma **allowlist** mútua. Apenas **delegados credenciados** cruzam a fronteira; a base de cada organização acessa o conteúdo da frente pelo **seu próprio** servidor (espelho). Os objetos trocados são os envelopes assinados do doc 03 — sem criptografia nova.

## Alternativas consideradas

- **Federação aberta (estilo ActivityPub/Fediverso)** — rejeitada: expõe metadados de participação por padrão e não reflete a lógica política (a frente exige acordo prévio entre direções).
- **Servidor único multi-organização** — rejeitada: viola P6 (uma organização, um servidor) e concentra o risco de apreensão.
- **Sem federação (organizações isoladas)** — rejeitada: impossibilitaria frentes comuns, um requisito explícito.

## Consequências

- (+) A base de membros de uma organização nunca é exposta à outra (só delegados credenciados atravessam).
- (+) O acordo político precede o peering técnico — o software reflete a prática da frente única.
- (+) Reuso integral do modelo de envelopes (doc 03); federação é transporte, não nova cripto.
- (−) Não há descoberta automática de pares; cada frente exige configuração deliberada (custo aceito, é o objetivo).
- (−) Federação multilateral (3+) e revogação de credencial ficam como problemas em aberto (doc 04 §8).
