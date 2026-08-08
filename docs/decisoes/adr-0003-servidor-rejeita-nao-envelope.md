# ADR-0003 — O servidor rejeita tudo que não seja envelope válido (com uma exceção)

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [03 §6](../03-arquitetura-criptografica.md), [02 §2.4](../02-modelo-de-dominio.md) (I7) |

## Contexto

O princípio do servidor cego (P2) exige que o servidor "obrigue tudo a ser criptografado". Um servidor não consegue, porém, **provar** que uma sequência de bytes opaca é realmente um texto cifrado. É preciso definir o que, concretamente, o servidor aceita ou rejeita — e ser honesto sobre a garantia que isso oferece.

## Decisão

O servidor aceita apenas objetos com a forma de **envelope** (doc 03 §6) e os valida **estruturalmente sem decifrar**: schema CBOR correto, assinatura do remetente conferida, remetente autorizado no organismo-destino (ACL), época de chave corrente e tamanhos canônicos (com padding em buckets). Qualquer objeto que falhe é **rejeitado**. A **única exceção formal** é a `Publicacao` de escopo `publico`, assinada mas legível por desenho (para alcançar simpatizantes e o exterior).

## Alternativas consideradas

- **Confiar que o cliente cifra, sem validação no servidor** — rejeitada: não impõe nada; qualquer cliente adulterado despejaria plaintext e o servidor o armazenaria.
- **Tentar provar que o corpo é cifrado** (heurísticas de entropia como garantia) — rejeitada como *garantia*: é indecidível em geral. Mantida apenas como **defesa em profundidade** (higiene), explicitamente não como prova.

## Consequências

- (+) Nenhum objeto malformado ou não-cifrado é aceito; ACL e autoria verificáveis sem quebrar o sigilo.
- (+) Regra clara e única de exceção (publicação pública), rastreável no invariante I7.
- (−) **Honestidade obrigatória:** a confidencialidade real depende do **cliente livre e auditável**, não de uma prova do servidor. O documento e a UI não podem prometer o contrário.
- (−) `remetente` e `organismo_destino` no cabeçalho em claro são metadados expostos ao servidor (mitigações no doc 03 §9 e doc 06).
