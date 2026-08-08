# Glossário

Vocabulário único do projeto. **Regra editorial: nenhum documento introduz um termo de domínio novo sem registrá-lo aqui, no mesmo commit.** Cada termo aponta para o documento que o define em profundidade.

Convenção: termos de domínio em português; primitivas criptográficas mantêm o nome técnico consagrado (em inglês quando não há tradução de uso corrente).

| Termo | Definição curta | Referência |
|---|---|---|
| **ADR** | Registro de Decisão de Arquitetura: documento curto que fixa uma decisão, as alternativas avaliadas e as consequências. | [docs/decisoes/](docs/decisoes/) |
| **Célula** | Organismo de base: grupo pequeno de militantes (referência: 3–15) vinculado a um local de trabalho, território ou setor; delibera, elege delegados e coleta cotização. Todo militante pertence a exatamente uma célula-base. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Centralismo democrático** | Método decisório: liberdade de discussão + unidade de ação; eleição de baixo para cima; decisões de instâncias superiores vinculam as inferiores; prestação de contas periódica. | [doc 01](docs/01-fundamentos-leninistas.md) |
| **Militante** | Usuário pleno: pertence a uma célula e participa das deliberações. Distinto de simpatizante. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Organismo** | Qualquer instância organizada da estrutura: célula, comitê, comissão, fração, congresso, buro. Os organismos formam uma árvore cuja raiz é a organização. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Organização** | Entidade política que roda um servidor; raiz da árvore de organismos; possui chave criptográfica própria e estatuto parametrizado. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Pseudônimo** | Nome fictício escolhido pelo usuário, único dentro do servidor, sem relação com a identidade civil. Único nome que o sistema conhece. | [doc 03](docs/03-arquitetura-criptografica.md) |
| **Servidor cego** | Propriedade de projeto (P2): o servidor armazena e roteia apenas envelopes cifrados e estrutura; não lê conteúdo e não guarda PII. | [doc 00](docs/00-visao.md), [doc 03](docs/03-arquitetura-criptografica.md) |
| **Simpatizante** | Usuário que acompanha as publicações públicas da organização sem pertencer a célula. | [doc 02](docs/02-modelo-de-dominio.md) |

*(Glossário vivo — termos são acrescentados conforme os documentos que os definem são escritos.)*
