# ADR-0007 — Licenciamento: CC BY-SA 4.0 para os documentos, AGPL-3.0 recomendada para o código

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-08 |
| **Documentos afetados** | [LICENSE](../../LICENSE), [README](../../README.md) |

## Contexto

O projeto é público (P6: software livre) e nesta fase é 100% documental. É preciso definir a licença dos documentos agora e registrar a recomendação para o código futuro, de modo coerente com a natureza da ferramenta (infraestrutura para organizações que precisam confiar e auditar o que rodam).

## Decisão

- **Documentação:** **Creative Commons Attribution-ShareAlike 4.0 (CC BY-SA 4.0)** — permite reuso e adaptação com atribuição, exigindo que derivados mantenham a mesma licença (copyleft de conteúdo).
- **Código (recomendação para a fase de implementação):** **AGPL-3.0** — copyleft forte que **cobre o uso em rede**: quem opera uma versão modificada do servidor para terceiros deve disponibilizar o código correspondente. Isso é diretamente relevante a um software cuja confiança depende de o cliente e o servidor serem os auditados (doc 06, premissa §3).

## Alternativas consideradas

- **Código sob MIT/Apache (permissiva)** — rejeitada como recomendação: permitiria versões fechadas do servidor, minando a garantia de que se roda o código auditado.
- **GPL-3.0 (sem "A")** — mais fraca para software de rede: não obriga a publicar modificações usadas apenas como serviço.
- **Documentação sob CC BY (sem SA)** — rejeitada: não garante que derivados permaneçam abertos.

## Consequências

- (+) Documentos e (futuramente) código permanecem abertos e auditáveis em toda a cadeia de derivados.
- (+) AGPL desincentiva forks fechados de servidor — coerente com o modelo de ameaças.
- (−) AGPL pode afastar integração com componentes de licença incompatível; avaliar caso a caso na implementação.
- (−) A recomendação de código é apenas isso nesta fase — a decisão definitiva será registrada quando o código existir.
