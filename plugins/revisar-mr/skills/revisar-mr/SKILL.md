---
name: revisar-mr
description: Revisa merge requests do Cérebro (Yii2) e do migrations-sgi seguindo os padrões do time. Funciona com link/!id do MR (lê e comenta no GitLab pelo Chrome) ou sem link, dentro do repositório (diff local da branch atual contra develop/master, relatório no chat). Use quando pedirem "revisar mr", "revisa esse merge", "code review do cérebro" ou rodarem /revisar-mr.
argument-hint: "[link do MR | !id | branch-base] [--comentar] [--migrations <link|caminho>]"
---

# Revisar MR

Revisão de código do Cérebro e do `df/migrations-sgi`. As regras estão em
[references/regras.md](references/regras.md). Leia o arquivo inteiro antes de revisar. As dicas de
API do GitLab ficam em [references/gitlab.md](references/gitlab.md).

## 1. Escolher o modo

Olhe o argumento recebido:

| Argumento | Modo |
|---|---|
| link `gitlab.sebrae.com.br/.../merge_requests/<id>`, `!<id>` ou só o número | **GitLab** |
| nome de branch (`develop`, `master`, `release/x`) | **Local**, com essa branch como base |
| vazio | **Local**, base automática (ver abaixo) |

Se o argumento for vazio e o diretório atual não for um repositório git, pergunte o link do MR.

## 2. Modo GitLab (MR pelo Chrome)

1. Use o Claude in Chrome. Se não houver navegador conectado ou o GitLab cair no SSO (AMEI), pare e
   avise. Não faça login.
2. Leia o MR, as discussões e o diff pela API (ver `references/gitlab.md`).
3. Antes de apontar escopo em MR grande, confira se os arquivos não são ruído de merge-base
   (`references/gitlab.md`, seção "MR com centenas de arquivos").
4. Procure o link de MR de migrations na descrição e nos comentários do autor e revise junto
   (regra 9).
5. Postar comentários:
   - com `--comentar`, ou quando chamado pela tarefa `olhar-mr`: crie as threads direto, ancoradas
     na linha (regra 3), sem perguntar;
   - sem `--comentar`: mostre o relatório e pergunte se deve postar. Só poste depois do "sim".
6. Nunca aprove o MR.

## 3. Modo Local (código no VS Code / terminal)

1. Base da comparação:
   - argumento com nome de branch: use ele;
   - sem argumento: branch atual `hotfix/*` → `master`; qualquer outra → `develop`.
   - Diga em uma linha qual base usou.
2. Atualize a base e pegue o diff a partir do merge-base (três pontos):
   ```bash
   git fetch origin <base>
   git diff origin/<base>...HEAD
   ```
   Mudanças ainda não commitadas (`git status --porcelain`, `git diff HEAD`) entram também, marcadas
   como "não commitado" no relatório.
3. Para conferir uso fora do diff (classe apagada, método renomeado, relação existente), use Grep e
   Read no próprio repositório. O código local já é a branch de origem.
4. Migrations:
   - `--migrations <link>`: leia o MR pelo Chrome como no modo GitLab;
   - `--migrations <caminho>`: repositório local do migrations-sgi, comparado com a **mesma base do
     Cérebro** (base `master` no Cérebro → `master` nas migrations):
     ```bash
     git -C <caminho> fetch origin <base>
     git -C <caminho> diff origin/<base>...HEAD
     ```
   - sem a opção, se o diff usa tabela/coluna nova (regra 9), pergunte onde estão as migrations. Se
     não houver resposta, registre "migrations não revisadas" no relatório.
5. Não comente no GitLab e não altere arquivos. No fim, ofereça corrigir os apontamentos se o
   usuário quiser.
6. Referencie cada apontamento como link `[arquivo.php:123](caminho/arquivo.php:123)`, para abrir
   direto no editor.

## 4. Relatório

Uma linha por apontamento (regra/tipo + o problema em poucas palavras). Formato:

```
!<id> ou <branch> → <base> — <autor> — <❌ com apontamentos | 🆗 pronto para aprovação | ✅ mergeado>
- <regra>: <problema> (<arquivo:linha>)
- <apontamento anterior não corrigido (thread de <pessoa>)>
- Migrations !<id> / <caminho>: <n> apontamentos — <resumo> (ou "sem migrations")
```

Formatação (regra 7) nunca entra como apontamento. Se só houver isso, o MR está 🆗.
