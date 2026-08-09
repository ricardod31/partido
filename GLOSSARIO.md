# Glossário

Vocabulário único do projeto. **Regra editorial: nenhum documento introduz um termo de domínio novo sem registrá-lo aqui, no mesmo commit.** Cada termo aponta para o documento que o define em profundidade.

Convenção: termos de domínio em português; primitivas criptográficas mantêm o nome técnico consagrado (em inglês quando não há tradução de uso corrente).

| Termo | Definição curta | Referência |
|---|---|---|
| **ADR** | Registro de Decisão de Arquitetura: documento curto que fixa uma decisão, as alternativas avaliadas e as consequências. | [docs/decisoes/](docs/decisoes/) |
| **Agitprop** | Papel no buro de um organismo responsável por agitação e propaganda: distribui o jornal, produz o boletim local e organiza a correspondência com a redação. | [doc 01](docs/01-fundamentos-leninistas.md) |
| **Bolchevização** | Campanha da Comintern (1924–25) que fixou a célula de empresa como unidade de base, no lugar da seção territorial. Origem histórica do princípio P1. | [doc 01](docs/01-fundamentos-leninistas.md) |
| **Buro** | O **conjunto de papéis de direção** de um organismo (secretário, tesoureiro, agitprop) — **não** é um subtipo de organismo. | [doc 01](docs/01-fundamentos-leninistas.md), [doc 02](docs/02-modelo-de-dominio.md) |
| **Bulletin board (quadro eleitoral)** | Registro público de uma eleição (nº de credenciais emitidas, censo congelado) que torna detectável a sobre-emissão por uma mesa. | [doc 03](docs/03-arquitetura-criptografica.md), [ADR-0006](docs/decisoes/adr-0006-voto-secreto-assinatura-cega.md) |
| **Credencial de membro anônima** | Prova em conhecimento zero (BBS+/KVAC) de que o remetente é membro autorizado de um organismo, sem revelar qual membro; permite ao servidor verificar ACL sem aprender o grafo. | [ADR-0008](docs/decisoes/adr-0008-mls-e-credenciais-anonimas.md) |
| **Célula** | Organismo de base: grupo pequeno de militantes (referência: 3–15) vinculado a um local de trabalho, território ou setor; delibera, elege delegados e coleta cotização. Todo militante pertence a exatamente uma célula-base. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Centralismo democrático** | Método decisório: liberdade de discussão + unidade de ação; eleição de baixo para cima; decisões de instâncias superiores vinculam as inferiores; prestação de contas periódica. No software, vira fluxo de dados (P4). | [doc 01](docs/01-fundamentos-leninistas.md) |
| **Comissão** | Organismo funcional permanente com escopo definido (ex.: finanças, ética, agitprop). | [doc 02](docs/02-modelo-de-dominio.md) |
| **Comitê** | Organismo dirigente de um escopo (local, regional), composto por mandatos eleitos pelas instâncias inferiores. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Congresso** | Organismo deliberativo supremo e temporário, composto por delegados credenciados, com pauta e prazo; elege a direção. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Correspondência** | Informe enviado de uma célula/organismo para a redação do jornal ou para a instância superior (fluxo ascendente do centralismo democrático). | [doc 02](docs/02-modelo-de-dominio.md) |
| **Cotização** | Contribuição financeira regular do membro, coletada pelo tesoureiro da célula; base material da independência da organização. | [doc 05](docs/05-financiamento.md) |
| **Deliberação** | Processo decisório de um organismo: proposta → discussão → emendas → votação → apuração → resolução. Pode ter voto aberto ou secreto. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Época de chave** | Versão corrente da chave simétrica de um organismo; é incrementada quando um membro entra ou sai, para controlar quem lê o quê. | [doc 03](docs/03-arquitetura-criptografica.md) |
| **Envelope** | Unidade de dados cifrada no cliente: cabeçalho mínimo em claro (para roteamento) + corpo autenticado e cifrado + assinatura do remetente. O servidor valida sua estrutura sem decifrar o corpo. | [doc 03](docs/03-arquitetura-criptografica.md) |
| **Escrutinador** | Membro da comissão eleitoral que detém uma parte da chave da urna; a urna só é aberta com o quórum de escrutinadores. | [doc 03](docs/03-arquitetura-criptografica.md) |
| **Fração** | Organismo transversal que coordena a atuação de militantes dentro de uma organização externa (sindicato, associação), sob a disciplina da organização-mãe. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Frente** | Acordo político explícito entre duas ou mais organizações para ação comum, preservando a independência de cada uma; realizada tecnicamente pela federação de servidores. | [doc 04](docs/04-federacao.md) |
| **Jornal** | Órgão editorial central da organização (e boletins por organismo): publicações assinadas que descem para a base e correspondências que sobem dela. Também serve de registro auditável. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Konspiratsiya** | Disciplina histórica de segurança clandestina (pseudônimos, need-to-know, separação de aparelhos); no projeto, é implementada por criptografia e minimização de metadados. | [doc 01](docs/01-fundamentos-leninistas.md) |
| **Mandato / Delegação** | Poder conferido por eleição de um organismo a um mandatário (delegado), com destino, prazo e revogabilidade (recall), sujeito a prestação de contas. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Militante** | Usuário pleno: pertence a uma célula e participa das deliberações. Distinto de simpatizante. | [doc 02](docs/02-modelo-de-dominio.md) |
| **MLS (Messaging Layer Security)** | Padrão de grupo E2E (RFC 9420) — árvore de ratchet assinada, confirmação de grupo, forward secrecy e post-compromise security; base do mecanismo de grupo dos organismos. | [ADR-0008](docs/decisoes/adr-0008-mls-e-credenciais-anonimas.md) |
| **Need-to-know** | Princípio de que cada membro acessa apenas o indispensável à sua tarefa; no software, é imposto por criptografia de grupo por organismo. | [doc 01](docs/01-fundamentos-leninistas.md), [doc 03](docs/03-arquitetura-criptografica.md) |
| **Organismo** | Qualquer instância organizada da estrutura: célula, comitê, comissão, fração, congresso, direção. Os organismos formam uma árvore cuja raiz é a organização. (O *buro* é o conjunto de papéis de direção de um organismo, não um subtipo.) | [doc 02](docs/02-modelo-de-dominio.md) |
| **Organização** | Entidade política que roda um servidor; raiz da árvore de organismos; possui chave criptográfica própria e estatuto parametrizado. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Pseudônimo** | Nome fictício escolhido pelo usuário, único dentro do servidor, sem relação com a identidade civil. Único nome que o sistema conhece. | [doc 03](docs/03-arquitetura-criptografica.md) |
| **Resolução** | Decisão registrada de um organismo (ata assinada), com escopo de vinculação; propaga-se para os feeds dos organismos subordinados. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Servidor cego** | Propriedade de projeto (P2): o servidor armazena e roteia apenas envelopes cifrados e estrutura; não lê conteúdo e não guarda PII. | [doc 00](docs/00-visao.md), [doc 03](docs/03-arquitetura-criptografica.md) |
| **Simpatizante** | Usuário que acompanha as publicações públicas da organização sem pertencer a célula. | [doc 02](docs/02-modelo-de-dominio.md) |
| **Voucher** | Código assinado pelo tesoureiro da célula, emitido contra pagamento em espécie, que o membro resgata como "cota paga"; permite anonimato do pagador com prestação de contas agregada. | [doc 05](docs/05-financiamento.md) |

*(Glossário vivo — termos são acrescentados conforme os documentos que os definem são escritos.)*
