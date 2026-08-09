# ADR-0004 — Criptografia de grupo por época simétrica no MVP; MLS como evolução

| | |
|---|---|
| **Status** | **substituída por [ADR-0008](adr-0008-mls-e-credenciais-anonimas.md)** (2026-08-09) |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [03 §7](../03-arquitetura-criptografica.md), [02 §2.2](../02-modelo-de-dominio.md) (I9) |

> **Aviso de substituição (2026-08-09).** A revisão crítica ([revisao-critica.md](../revisao-critica.md) §2-B) mostrou que a distribuição de chave de época por `crypto_box_seal` é anônima e não-autenticada (o servidor pode fabricar pacotes de chave e equivocar membros), que o modelo não tem *post-compromise security* (comprometer uma `pk_enc` dá leitura permanente do futuro) e que o avanço de época não é autenticado (downgrade). Esta decisão foi **substituída** por [ADR-0008](adr-0008-mls-e-credenciais-anonimas.md), que adota MLS (RFC 9420). O texto abaixo é mantido como registro histórico.

## Contexto

Cada organismo é um compartimento criptográfico (M5, need-to-know): só seus membros leem seu conteúdo, e a saída de um membro deve cortar seu acesso futuro (I9). Ao mesmo tempo, uma organização política precisa de **memória durável** (atas, resoluções, jornal legíveis ao longo do tempo). É preciso escolher o mecanismo de chave de grupo do MVP.

## Decisão

No MVP, cada organismo tem uma **chave simétrica por época**. A chave da época corrente é distribuída a cada membro por **sealed box** (`crypto_box_seal`) para sua `pk_enc`. Entrada ou saída de membro **incrementa a época** e redistribui uma nova chave: quem sai não lê o futuro; quem entra não lê o passado por padrão (need-to-know). **MLS (RFC 9420)** fica registrado como a evolução natural, com o formato de envelope desenhado para permitir a migração.

## Alternativas consideradas

- **MLS desde o MVP** — adiada, não rejeitada: dá melhor forward secrecy e *post-compromise security*, mas com complexidade de implementação alta; desproporcional para o primeiro ciclo.
- **Double ratchet (estilo Signal) para tudo** — rejeitada para organismos: é ótimo para conversa efêmera entre dois, ruim para grupo com memória durável e composição estável.
- **Cifrar por-destinatário cada mensagem** (sem chave de grupo) — rejeitada: custo O(n) por mensagem e sem noção de "conteúdo do organismo".

## Consequências

- (+) Simples, auditável, e adequado à necessidade de registros duráveis por época.
- (+) I9 satisfeita: rotação obrigatória na mudança de composição.
- (−) *Forward secrecy* fraca dentro de uma mesma época (a chave da época protege todo o conteúdo dela); é um tradeoff consciente a favor da memória organizacional.
- (−) Distribuição de chave depende de os membros terem `pk_enc` válidas e online em algum momento para receber o pacote.
