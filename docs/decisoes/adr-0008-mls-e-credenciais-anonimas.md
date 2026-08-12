# ADR-0008 — MLS como base de grupo e credenciais de membro anônimas

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-09 |
| **Substitui** | [ADR-0004](adr-0004-cripto-de-grupo-por-epoca.md) (chave de época por sealed box) |
| **Documentos afetados** | [03 §6/§7/§9](../03-arquitetura-criptografica.md), [00 P2](../00-visao.md), [06 A3/A5](../06-modelo-de-ameacas.md) |
| **Origem** | [revisao-critica.md](../revisao-critica.md) §2-B, §2-C |

## Contexto

A revisão crítica derrubou dois pilares do desenho original de grupo (ADR-0004) e um do envelope:

1. **Distribuição de chave por `crypto_box_seal` é anônima e não-autenticada.** Como a `pk_enc` é pública, qualquer parte — em especial o servidor — pode fabricar um pacote de chave de época e **equivocar** membros (entregar chaves diferentes a membros diferentes), inserir-se como leitor, ou fazer *downgrade* de época. Não há confirmação de que o grupo compartilha a mesma chave.
2. **Ausência de *post-compromise security* (PCS) e forward secrecy.** Como a `pk_enc` de um membro não muda na rotação de composição, comprometer uma chave privada (adversário A4, previsto no modelo) dá leitura **permanente** de todo o conteúdo futuro do organismo.
3. **O grafo de filiação — o ativo nº 2 — trafega em claro.** `remetente` + `organismo_destino` + o `papel` (necessário à ACL) entregam ao servidor, em tempo real, o organograma pseudônimo e a hierarquia. É exatamente "o primeiro alvo de qualquer repressão" ([doc 00 §1](../00-visao.md)).

## Decisão

**(a) Grupo por MLS (RFC 9420).** O mecanismo de grupo de cada organismo passa a ser o **Messaging Layer Security** — árvore de ratchet assinada, `tree_hash` e `confirmation_tag` (todos os membros confirmam a mesma visão de chave e composição), com PCS e forward secrecy nativos, adição/remoção em lote (`Commit`) e avanço de época (`epoch`) **autenticado**. Isso elimina de uma vez a equivocação (1), a falta de PCS (2) e o downgrade de época.

**(b) ACL por credencial de membro anônima (BBS+/KVAC).** Para o servidor autorizar um envelope (verificar que o remetente é membro do organismo com papel apto) **sem aprender qual membro é**, adota-se uma **credencial anônima** (assinatura BBS+ ou KVAC estilo *Signal private groups*): o cliente prova em conhecimento zero "sou membro autorizado do organismo O para o tipo T". O `remetente` nominal sai do cabeçalho em claro. Isso ataca simultaneamente a exposição do grafo (3), a ausência de *deniability* (assinatura nominal era prova não-repudiável) e o problema do "sealed sender".

## Alternativas consideradas

- **Endurecer o modelo de época** (pacote de chave assinado + confirmation tag + transcrição) — melhora, mas reimplementa mal o que o MLS já provê revisado e padronizado (P7); rejeitada em favor do padrão.
- **Sealed sender à la Signal** para o grafo — **insuficiente** aqui, porque o servidor *precisa* verificar ACL; sealed sender esconde o remetente mas não permite a autorização. Credencial anônima resolve os dois.
- **Manter assinatura nominal no envelope** — rejeitada: é prova não-repudiável de autoria (passivo criminal numa apreensão) e expõe o grafo.

## Consequências

- (+) Equivocação de servidor, ausência de PCS/FS e downgrade de época deixam de existir (propriedades do MLS).
- (+) O servidor autoriza sem aprender identidade nem grafo por mensagem; ganha-se *deniability*.
- (+) Alinha o projeto a um padrão auditado (RFC 9420) em vez de construção própria (P7).
- (−) **Complexidade de implementação alta** — MLS e provas ZK BBS+/KVAC exigem bibliotecas maduras e cuidado; é o maior custo de engenharia do projeto.
- (−) A "memória durável" (atas legíveis no tempo) passa a exigir **camada de arquivo explícita** (registro re-cifrado sob chave de arquivo do organismo) separada do transporte com FS — durabilidade do texto e higiene de chave tornam-se ortogonais, como devem ser.
- (−) A revogação de credencial anônima (quando um membro sai) exige mecanismo próprio (epoch/accumulator) — item de projeto do reprojeto do [doc 03](../03-arquitetura-criptografica.md).
