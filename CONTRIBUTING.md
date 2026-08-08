# Como contribuir

Este repositório está em **fase de desenho**: contém apenas especificações, nenhum código. As contribuições mais valiosas agora são de revisão e crítica.

## O que ajuda nesta fase

1. **Revisão conceitual** — o mapeamento histórico→software ([doc 01](docs/01-fundamentos-leninistas.md)) está correto? O modelo de domínio ([doc 02](docs/02-modelo-de-dominio.md)) cobre a prática real de organizações de base?
2. **Revisão de segurança** — leitura crítica dos [docs 03](docs/03-arquitetura-criptografica.md) e [06](docs/06-modelo-de-ameacas.md) por quem tem experiência em criptografia aplicada. Procuramos especialmente: construções sem nome padrão, promessas que os mecanismos não sustentam, metadados não declarados.
3. **Revisão de texto** — clareza das seções `[conceitual]` para leitores não-técnicos.

## Regras editoriais

- Todo o repositório é em **pt-BR**; nomes de arquivo sem acentos, em kebab-case.
- Seções são marcadas `[conceitual]` ou `[técnico]` (ver convenção no [doc 00](docs/00-visao.md)).
- Nenhum termo de domínio novo sem entrada no [GLOSSARIO.md](GLOSSARIO.md) no mesmo commit.
- Decisões de arquitetura mudam por **ADR** ([docs/decisoes/](docs/decisoes/)), nunca por edição silenciosa: um ADR aceito só é revertido por outro ADR que o substitua.
- Diagramas são mermaid embutidos no markdown; rótulos com acentos ou parênteses sempre entre aspas.
- Criptografia: só primitivas padrão com referência (RFC ou libsodium) — princípio P7. Sugestões de esquema "caseiro" serão recusadas.

## Fluxo

Issues para discussão; pull requests para mudanças de texto. PRs que alterem decisões registradas devem incluir o ADR correspondente.

## Aviso

Projeto de engenharia para organização política **legal** (liberdade de associação). Não abra issues contendo dados de pessoas ou organizações reais, nem detalhes operacionais de grupos existentes.

*(Um workflow de CI para lint de markdown e validação de mermaid é desejável, mas fica para quando o repositório sair da fase puramente documental.)*
