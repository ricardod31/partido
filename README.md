# partido

**Software livre de organização política de base** — células, instâncias, deliberações, eleições, jornal e finanças — com criptografia de ponta a ponta por padrão, identidade por par de chaves e federação entre organizações.

> **Status: fase de desenho.** Este repositório contém apenas especificações e estudos; **nenhum código ainda**. O objetivo desta fase é desenhar o sistema: estudar a tradição organizativa que o inspira, definir o modelo de domínio e especificar a arquitetura criptográfica.

## A ideia em um parágrafo

Organizações políticas hoje se organizam em plataformas comerciais vigiadas e sem forma organizativa. Este projeto desenha a alternativa: um sistema onde **a célula é a unidade fundamental** (P1), **o servidor é cego quanto ao conteúdo** — só armazena envelopes cifrados no cliente e não guarda nome civil, e-mail ou telefone (P2; o que permanece visível como estrutura, e a análise LGPD do grafo pseudônimo, estão declarados no [modelo de ameaças](docs/06-modelo-de-ameacas.md) e em [revisao-critica-2.md](docs/revisao-critica-2.md)), **a identidade é um par de chaves com pseudônimo** (P3), o método decisório é o **centralismo democrático** transformado em fluxo de dados (P4), a comunicação central é um **jornal** — o organizador coletivo (P5), e **cada organização roda seu próprio servidor**, federando com outras apenas por acordo político explícito (P6). Os princípios completos estão no [documento de visão](docs/00-visao.md).

## Mapa de leitura

| Documento | Pergunta que responde | Público |
|---|---|---|
| [00 — Visão](docs/00-visao.md) | O que é o projeto e quais princípios o regem? | todos |
| [01 — Fundamentos leninistas](docs/01-fundamentos-leninistas.md) | Como os partidos leninistas se organizavam, e como isso vira software? | todos |
| [02 — Modelo de domínio](docs/02-modelo-de-dominio.md) | Quais são as entidades do sistema (usuário, célula, organismo, mandato, jornal, deliberação) e suas regras? | todos / técnico |
| [03 — Arquitetura criptográfica](docs/03-arquitetura-criptografica.md) | Como funcionam identidade, autenticação, envelopes cifrados, grupos e voto secreto? | técnico (com seções conceituais) |
| [04 — Federação](docs/04-federacao.md) | Como duas organizações formam uma frente comum, servidor a servidor? | técnico (com seções conceituais) |
| [05 — Financiamento](docs/05-financiamento.md) | Como receber contribuições pequenas com anonimato do contribuinte — e o que é honestamente possível? | todos |
| [06 — Modelo de ameaças](docs/06-modelo-de-ameacas.md) | Contra o que o sistema protege e contra o que ele **não** protege? | todos |
| [07 — Páginas e experiência](docs/07-paginas/README.md) | Quais páginas o sistema tem, como se navega e qual é o design system? | todos / técnico |
| [Decisões (ADRs)](docs/decisoes/) | Por que cada decisão de arquitetura foi tomada? | técnico |
| [Glossário](GLOSSARIO.md) | O que cada termo significa? | todos |

Seções são marcadas **[conceitual]** (para militantes sem formação técnica) ou **[técnico]** (especificação para engenheiros).

## Avisos

- **Legalidade e neutralidade.** Este é um projeto de engenharia para organização política **legal**, exercício da liberdade de associação (no Brasil, art. 5º, XVII–XXI da Constituição Federal). O estudo histórico ([doc 01](docs/01-fundamentos-leninistas.md)) é funcional-organizativo, não endosso de eventos históricos. O repositório não contém dados de pessoas ou organizações reais.
- **Honestidade sobre segurança.** Nenhum software torna alguém "anônimo" ou "seguro" em absoluto. O [modelo de ameaças](docs/06-modelo-de-ameacas.md) declara explicitamente o que o desenho protege e o que não protege.

## Contribuindo

Ver [CONTRIBUTING.md](CONTRIBUTING.md). Nesta fase, as contribuições mais valiosas são revisão conceitual, revisão de segurança e revisão de texto.

## Licença

Documentação sob [CC BY-SA 4.0](LICENSE). Código futuro: recomendação registrada de AGPL-3.0 (ver [ADR-0007](docs/decisoes/adr-0007-licenciamento.md)).
