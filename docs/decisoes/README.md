# Decisões de arquitetura (ADRs)

Cada decisão de projeto significativa é registrada como um **ADR** (Architecture Decision Record): contexto, decisão, alternativas consideradas e consequências (incluindo os custos). Uma decisão registrada só muda por outro ADR que a substitua — nunca por edição silenciosa (ver [CONTRIBUTING](../../CONTRIBUTING.md)).

| ADR | Decisão | Documento |
|---|---|---|
| [0001](adr-0001-identidade-por-par-de-chaves.md) | Identidade por par de chaves, sem PII no servidor | [03](../03-arquitetura-criptografica.md) |
| [0002](adr-0002-autenticacao-assinatura-de-nonce.md) | Autenticação por assinatura de nonce (não por oráculo de decifração) | [03](../03-arquitetura-criptografica.md) |
| [0003](adr-0003-servidor-rejeita-nao-envelope.md) | O servidor rejeita tudo que não seja envelope válido (uma exceção) | [03](../03-arquitetura-criptografica.md) |
| [0004](adr-0004-cripto-de-grupo-por-epoca.md) | Criptografia de grupo por época simétrica no MVP; MLS como evolução | [03](../03-arquitetura-criptografica.md) |
| [0005](adr-0005-federacao-por-acordo-bilateral.md) | Federação por acordo bilateral explícito (allowlist) | [04](../04-federacao.md) |
| [0006](adr-0006-voto-secreto-assinatura-cega.md) | Voto secreto por assinatura cega + urna de chave dividida | [03](../03-arquitetura-criptografica.md) |
| [0007](adr-0007-licenciamento.md) | Licenciamento (CC BY-SA 4.0 docs / AGPL-3.0 código) | [README](../../README.md) |

Novos ADRs seguem o [template](template.md) e recebem o próximo número livre.
