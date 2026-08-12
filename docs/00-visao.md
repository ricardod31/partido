# 00 — Visão do projeto

| | |
|---|---|
| **Status** | rascunho |
| **Última atualização** | 2026-08-08 |
| **Depende de** | — |
| **Público** | todos |

> Convenção de leitura: seções marcadas **[conceitual]** são escritas para militantes e dirigentes sem formação técnica; seções marcadas **[técnico]** são especificação para engenheiros. Termos do projeto estão definidos no [GLOSSARIO.md](../GLOSSARIO.md).

## 1. O problema [conceitual]

Organizações políticas de base — partidos, coletivos, movimentos — hoje se organizam sobre plataformas comerciais que não foram feitas para isso:

1. **Vigilância estrutural.** Grupos de WhatsApp, Telegram e planilhas no Google expõem a plataformas privadas (e, por extensão, a quem as pressione) o ativo mais sensível de qualquer organização: **quem é membro, de qual instância, e o que se discute**. O histórico da organização política mostra que a lista de filiação é o primeiro alvo de qualquer repressão.
2. **Ausência de forma organizativa.** Um grupo de mensagens é uma multidão num salão: não tem célula, não tem instância, não tem mandato, não tem deliberação com quórum, não tem prestação de contas. As ferramentas moldam a prática — e ferramentas sem estrutura produzem organizações sem estrutura.
3. **Dependência de terceiros.** A organização não controla sua infraestrutura: contas são banidas, grupos são derrubados, dados são entregues.

Este projeto desenha um **software livre de organização política** que resolve os três problemas ao mesmo tempo: estrutura organizativa real (inspirada no modelo celular leninista, a tradição organizativa mais estudada e testada do movimento operário), criptografia de ponta a ponta por padrão (o servidor não lê nada), e auto-hospedagem (cada organização roda seu próprio servidor).

## 2. Princípios de projeto

Os princípios abaixo são numerados e **citáveis** — os demais documentos referem-se a eles como P1..P8. Qualquer decisão de design que contrarie um princípio precisa de um ADR (registro de decisão) justificando a exceção.

- **P1 — A célula é a unidade fundamental.** O sistema é organizado em torno de células (grupos pequenos de militantes) e dos organismos que elas constroem — não em torno de indivíduos soltos nem de "grupos" amorfos. Toda estrutura superior deriva, por eleição, das células.
- **P2 — O servidor é cego quanto ao *conteúdo* (não quanto à estrutura).** Todo conteúdo é criptografado no cliente, de ponta a ponta; o servidor armazena e roteia envelopes cifrados que **rejeita** quando malformados e **não consegue ler** quando válidos, e não guarda nenhum PII por construção. **Honestidade necessária (ver [revisao-critica.md](revisao-critica.md) e [doc 06](06-modelo-de-ameacas.md)):** "cego" vale para o *conteúdo*, não para o *grafo* — na versão inicial o servidor via, em claro, quem é membro de qual organismo e com qual papel. Isso é tratado como falha a corrigir: adotam-se **credenciais de membro anônimas** ([ADR-0008](decisoes/adr-0008-mls-e-credenciais-anonimas.md)) para que a autorização não exponha a filiação. E o "cego" só vale enquanto o **cliente** roda o código auditado (build reprodutível — [doc 06 A7](06-modelo-de-ameacas.md)).
- **P3 — Identidade é um par de chaves.** Um usuário é um par de chaves criptográficas mais um pseudônimo. Quem possui a chave privada **é** o usuário; não há senha recuperável, não há cadastro civil. A prova de identidade é sempre uma assinatura.
- **P4 — Centralismo democrático como fluxo de dados *com disciplina*.** O método decisório da tradição leninista vira arquitetura de informação: **eleição e informes sobem**, **decisões vinculantes descem**, **discussão precede o voto**, **prestação de contas é periódica** (mandatos expiram e são revogáveis). Para que "vinculante" não seja palavra vazia, a força das resoluções e as **sanções** (censura, afastamento, desligamento) são **consequência de deliberação com quórum** ([ADR-0009](decisoes/adr-0009-disciplina-como-deliberacao.md); invariante I11) — não um botão que um papel aciona sozinho. Consequência assumida: o software **não é neutro** — ele grava um modelo de governança democrático-centralista (ver [doc 01 C.6](01-fundamentos-leninistas.md)).
- **P5 — O jornal é o organizador coletivo.** A comunicação central da organização é um órgão editorial (jornal), com fluxo bidirecional: correspondências das células sobem para a redação; a publicação assinada desce para todas as células. O jornal também serve de **registro auditável** da linha e das resoluções da organização.
- **P6 — Uma organização é um servidor.** Cada organização hospeda (ou contrata sob seu controle) seu próprio servidor. Servidores federam **somente por acordo político explícito** entre organizações ("frentes"), nunca por federação aberta automática.
- **P7 — Só criptografia padrão de mercado.** Nenhuma primitiva criptográfica é inventada aqui. Tudo se apoia em algoritmos padronizados (RFCs) e implementações auditadas (libsodium como referência). Onde houver tentação de "criar um esquema", a resposta é citar um padrão ou declarar o problema em aberto.
- **P8 — O software incentiva a segurança operacional, mas não a substitui.** Criptografia não protege contra um infiltrado que é membro legítimo, nem contra um dispositivo apreendido desbloqueado. O software embute os incentivos certos (pseudônimos, compartimentação, need-to-know por padrão) e educa no onboarding — e os documentos deste projeto são honestos sobre o que **não** é protegido.

## 3. O que o sistema é [conceitual]

Em uma frase: **uma infraestrutura de organização — células, instâncias, deliberações, eleições, jornal e finanças — onde o servidor enxerga apenas a estrutura, nunca o conteúdo, e cada pessoa é um pseudônimo com uma chave.**

Os blocos principais (detalhados nos documentos seguintes):

| Bloco | O que faz | Documento |
|---|---|---|
| Identidade | par de chaves + pseudônimo; autenticação por desafio-resposta | [03](03-arquitetura-criptografica.md) |
| Organismos | células, comitês, comissões, frações, congressos em árvore | [02](02-modelo-de-dominio.md) |
| Deliberação | proposta → discussão → voto (aberto ou secreto) → resolução vinculante | [02](02-modelo-de-dominio.md), [03](03-arquitetura-criptografica.md) |
| Mandatos | delegados eleitos, revogáveis, com prestação de contas | [02](02-modelo-de-dominio.md) |
| Jornal | órgão editorial central + boletins por organismo + correspondência das células | [02](02-modelo-de-dominio.md) |
| Federação | frentes comuns entre organizações, servidor a servidor | [04](04-federacao.md) |
| Finanças | cotização com anonimato do contribuinte e prestação de contas agregada | [05](05-financiamento.md) |

## 4. Escopo desta fase / fora de escopo

**Nesta fase (desenho):**

- Estudo dos fundamentos organizativos leninistas e seu mapeamento para o software ([doc 01](01-fundamentos-leninistas.md));
- Modelo de domínio completo ([doc 02](02-modelo-de-dominio.md));
- Especificação da arquitetura criptográfica e de federação ([docs 03](03-arquitetura-criptografica.md) e [04](04-federacao.md));
- Estudo de opções de financiamento ([doc 05](05-financiamento.md));
- Modelo de ameaças ([doc 06](06-modelo-de-ameacas.md));
- Registros de decisão ([docs/decisoes/](decisoes/)).

**Fora de escopo nesta fase:**

- Qualquer código de produção, escolha final de stack, banco de dados ou framework;
- Design de interface (UI/UX);
- Especificação de API REST/RPC campo a campo (os fluxos e formatos de envelope são especificados; endpoints são esboçados apenas onde necessário para o raciocínio);
- Governança do projeto de software em si (mantenedores, roadmap).

## 5. Não-objetivos permanentes

- **Não é rede social.** Não há perfil público de indivíduo, feed global, seguidores ou métricas de engajamento.
- **Não é ferramenta de anonimato absoluto.** O sistema protege conteúdo e minimiza metadados, mas participação em uma organização deixa rastros estruturais (ver [doc 06](06-modelo-de-ameacas.md)); quem precisa de negação plausível total precisa de mais do que um software.
- **Não substitui o juízo político.** Admissão, desligamento e disciplina são processos sociais; o software registra e protege, não decide.

## 6. Decisões em aberto

- Nome definitivo do projeto (usa-se "partido" como nome de trabalho do repositório).
- Stack de implementação (fase seguinte; o desenho é agnóstico, com libsodium como referência de biblioteca criptográfica).

## Referências

- Documentos deste repositório citados acima.
- Princípios inspirados em: prática organizativa leninista (doc 01); arquitetura de sistemas E2E modernos — Signal, MLS (RFC 9420) — citados no doc 03.
