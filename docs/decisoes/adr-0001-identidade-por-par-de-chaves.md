# ADR-0001 — Identidade por par de chaves, sem PII no servidor

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [03 §3](../03-arquitetura-criptografica.md), [02 §2.1](../02-modelo-de-dominio.md) |

## Contexto

O ativo mais sensível de uma organização política é a identidade de seus membros (P8, doc 06). O sistema precisa identificar de forma estável quem é quem — para ACL, autoria e voto — sem jamais custodiar dados que, se apreendidos, exponham pessoas reais (P2, P3).

## Decisão

A identidade de um usuário é um **par de chaves** gerado no cliente: um par **Ed25519 de identidade** (assina) e um par **X25519 de cifra** (recebe conteúdo), pares **separados**, com a chave de cifra **certificada** pela de identidade. O identificador é auto-certificante: `user_id = multibase(BLAKE2b-256(pk_id))`. O servidor guarda **apenas** chaves públicas, pseudônimo, nível e estado. Não existe campo para nome real, e-mail ou telefone — a ausência de PII é estrutural.

## Alternativas consideradas

- **Conta com login/senha + e-mail** — rejeitada: cria PII custodiada, ponto único de comprometimento, superfície de recuperação atacável.
- **Uma única chave para assinar e cifrar** — rejeitada: viola a separação de propósitos; impede rotação independente da chave de cifra.
- **Identidade federada externa (OAuth/SSO)** — rejeitada: entrega a identidade a um terceiro; incompatível com P2/P6.

## Consequências

- (+) Servidor apreendido não revela identidades civis; nada de PII a vazar.
- (+) `user_id` auto-certificante impede o servidor de "trocar" identidades sem detecção.
- (−) **Perda da chave = perda da conta** (doc 03 §3.2): sem recuperação pelo servidor. Mitigação: backup por frase mnemônica; recuperação social como evolução futura.
- (−) Onboarding exige gerar e guardar chave com segurança — carga de usabilidade que o cliente precisa endereçar bem.
