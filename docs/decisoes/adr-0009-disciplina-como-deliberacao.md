# ADR-0009 — Disciplina e força vinculante como consequência de deliberação registrada

| | |
|---|---|
| **Status** | aceita |
| **Data** | 2026-08-09 |
| **Documentos afetados** | [00 P4](../00-visao.md), [02 §2.5/§2.6, I3, I11](../02-modelo-de-dominio.md), [01 C.1/C.2](../01-fundamentos-leninistas.md) |
| **Origem** | [revisao-critica.md](../revisao-critica.md) §2-A, §3 (achado 8) |

## Contexto

A revisão crítica mostrou que o centralismo democrático (P4) — o "coração" declarado do projeto — não tinha implementação do **polo centralista**: (1) resoluções "vinculantes" (I8/I10) não tinham consequência alguma para quem as ignorasse (a Parte C.2 do doc 01 dizia que o software "não decide sanções"); e (2) a exclusão criptográfica de um membro (mudar `estado→desligado`, disparando rotação de época) podia, no modelo, ser acionada por uma única pessoa — uma **expulsão unipessoal de facto**, um "superusuário oculto" que contradiz I6. Havia ainda contradição interna: C.1 afirmava que o sistema implementa "unidade de ação", enquanto C.2 removia todo meio de exigi-la.

## Decisão

**Sanção e força vinculante são atos de deliberação, não mutações de estado avulsas.** Concretamente:

1. **Nova invariante I11 — sanção só por deliberação.** Qualquer transição de `estado` de membro que restrinja direitos (censura, afastamento, `desligado`) só é válida como **consequência de uma deliberação com quórum** do organismo competente — espelhando I3 ("poder só por eleição"). A exclusão criptográfica (rotação que corta o membro) é *executada* pelo sistema, mas *decidida* pela deliberação.
2. **Força vinculante é modelada.** Uma `Resolução` carrega seu `escopo_vinculacao`, e o descumprimento por um organismo/membro é, ele próprio, matéria de deliberação (podendo gerar sanção via I11). "Vinculante" deixa de ser rótulo e passa a ter um caminho de execução (deliberação → sanção), sem que o software "julgue" — ele registra e executa decisões humanas registradas.
3. **O software embute um modelo de governança.** Assume-se, no [doc 00](../00-visao.md)/[doc 01 C.5](../01-fundamentos-leninistas.md), que o sistema **não é neutro**: ele grava um modelo democrático-centralista (I1–I11). A moldura "ferramenta que serve a qualquer organização" é substituída por essa declaração explícita.

## Alternativas consideradas

- **Assumir "deliberativo-delegativo"** (declarar que o projeto não implementa disciplina leninista e renomear P4) — alternativa honesta, **não escolhida**: descaracterizaria a proposta central do projeto (organização celular com unidade de ação).
- **Deixar a disciplina fora do software (processo puramente social)** — status quo rejeitado: deixava "vinculante" sem mecanismo e a exclusão como botão unipessoal (fere I6).

## Consequências

- (+) O centralismo democrático passa a ter os dois polos: eleição sobe (I3), decisão vinculante desce **com consequência** (I11).
- (+) Fecha o "superusuário oculto": ninguém exclui sozinho; exclusão é consequência de deliberação com quórum.
- (−) Exige modelar processos disciplinares (tipos de sanção, quórum qualificado, direito de defesa?) — carga de design adicional no [doc 02](../02-modelo-de-dominio.md).
- (−) O software assume abertamente um viés de governança; a moldura de "neutralidade" é abandonada (o que é mais honesto, mas exige tratar o uso dual — ver [doc 06](../06-modelo-de-ameacas.md), nota de uso dual).
