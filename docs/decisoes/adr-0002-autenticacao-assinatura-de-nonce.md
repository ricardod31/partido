# ADR-0002 — Autenticação por assinatura de nonce (não por oráculo de decifração)

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [03 §5](../03-arquitetura-criptografica.md) |

## Contexto

O sistema autentica usuários provando **posse da chave privada de identidade**, sem senha custodiada. A intuição inicial do projeto foi: "o servidor cifra um segredo com a chave pública do usuário; ele devolve o segredo decifrado". É preciso decidir o protocolo de desafio-resposta correto.

## Decisão

A autenticação é **desafio-resposta por assinatura de nonce com separação de domínio**: o servidor envia um nonce de uso único (≥128 bits, expiração curta); o cliente **assina** `"partido-auth-v1" || id_servidor || nonce || timestamp` com `sk_id`; o servidor verifica com `pk_id`, invalida o nonce e emite um token de sessão curto. É o mesmo padrão de SSH e WebAuthn/FIDO2.

## Alternativas consideradas

- **Oráculo de decifração** ("cifra um segredo, devolve decifrado") — **rejeitada**. Transforma o cliente num oráculo: um servidor malicioso ou um MitM pode submeter *qualquer* ciphertext (inclusive mensagens reais capturadas) e obter o texto decifrado. Além disso, mistura a chave de cifra com a função de autenticação, violando a separação de chaves (ADR-0001).
- **Senha/token compartilhado** — rejeitada: cria segredo custodiável no servidor.

## Consequências

- (+) Não há oráculo de decifração; a chave de cifra nunca é usada para autenticar.
- (+) Separação de domínio impede que uma assinatura de login seja confundida com assinatura de voto, resolução ou envelope.
- (+) `id_servidor` no desafio impede reuso de resposta entre servidores (relevante na federação, doc 04).
- (−) Exige relógio razoavelmente sincronizado (uso de `timestamp` + janela de expiração do nonce).
