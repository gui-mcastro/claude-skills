# claude-skills

Skills de agente do Guilherme Moura de Castro.

| Skill | O que faz |
|---|---|
| [revisar-mr](plugins/revisar-mr/skills/revisar-mr/SKILL.md) | Code review de MRs do Cérebro (Yii2) e do `df/migrations-sgi`, pelo GitLab (Chrome) ou localmente (git diff). |

## Instalar no Claude Code

```
/plugin marketplace add gui-mcastro/claude-skills
/plugin install revisar-mr@gui-mcastro-skills
```

Uso: `/revisar-mr` (ou `/revisar-mr:revisar-mr` se houver outra skill com o mesmo nome). Para
atualizar: `/plugin marketplace update gui-mcastro-skills`.

Repositório privado: a máquina precisa ter acesso ao GitHub (`gh auth login` ou credencial git).

## Exemplos de uso

Formato: `/revisar-mr [link do MR | !id | branch-base] [--comentar] [--migrations <link|caminho>]`.

### Local: só o Cérebro

No Claude Code (VS Code ou terminal), com o repositório do Cérebro aberto na branch da feature:

```
/revisar-mr
```
Compara `origin/develop...HEAD` (ou `origin/master...HEAD` se a branch for `hotfix/*`) e inclui o
que ainda não foi commitado.

```
/revisar-mr master
```
Força a base `master` (ex.: correção que vai direto para produção sem ser `hotfix/*`).

```
/revisar-mr release/2026-10
```
Qualquer branch pode ser a base.

```
revisa minha branch antes de eu abrir o MR
```
Pedido em texto livre também carrega a skill, no modo local com base automática.

### Local: Cérebro + migrations

Deixe o repositório do `migrations-sgi` na branch da feature correspondente antes de rodar:

```bash
git -C C:/projetos/migrations-sgi checkout feature/suborigem-usuario
```

```
/revisar-mr --migrations C:\projetos\migrations-sgi
```
Cérebro e migrations comparados com `origin/develop`. Cruza tabelas/colunas das migrations com
models, `rules` e views (nome, nullable × `required`, tamanho × `max`).

```
/revisar-mr master --migrations C:\projetos\migrations-sgi
```
Os dois comparados com `origin/master`. A base das migrations é sempre a mesma do Cérebro.

```
/revisar-mr hotfix/ajuste-sac --migrations C:\projetos\migrations-sgi
```
Base explícita para os dois repositórios.

```
/revisar-mr --migrations https://gitlab.sebrae.com.br/df/migrations-sgi/-/merge_requests/5127
```
Cérebro local e migrations lidas do MR no GitLab (pelo Chrome). A base das migrations é a branch
de destino desse MR. Útil quando você não tem o `migrations-sgi` clonado.

```
/revisar-mr --migrations !5127
```
Mesmo caso, usando só o número do MR de migrations.

```
/revisar-mr
```
Sem `--migrations`: se o diff usar tabela ou coluna nova, a skill pergunta onde estão as
migrations. Sem resposta, o relatório sai com "migrations não revisadas".

### Local: depois do relatório

```
corrige os apontamentos 1 e 3
```
No modo local a skill não altera nada sozinha. Ela oferece corrigir no fim, e você escolhe o quê.

### GitLab: MR aberto

Precisa do Claude in Chrome logado no GitLab do Sebrae.

```
/revisar-mr https://gitlab.sebrae.com.br/df/cerebro/-/merge_requests/13155
```
Revisa o MR (e o de migrations linkado na descrição) e mostra o relatório. Pergunta antes de
postar as threads.

```
/revisar-mr !13155 --comentar
```
Posta as threads direto, ancoradas na linha, sem perguntar.

### Exemplo de relatório

```
feature/usuario-web-service → develop — Davi — ❌ com apontamentos
- Yii 5: idPanel 'clp-form-usuario-web-service-origem' deveria ser 'clp-origem-suborigem' (views/usuario-web-service/_form-origem.php:16)
- Repetição: SistemaOrigem::dropDownList inline na view; usar SistemaOrigem::dropDownListOrigem() (views/usuario-web-service/_form-origem.php:31)
- Migrations C:\projetos\migrations-sgi: 1 apontamento
  - Integridade: cd_suborigem NULL, mas o model exige 'required' (database/migrations/2026/10/2026_09_14_133522_create_git_usuario_web_service_suborigem_table.php:18)
```

## Outros agentes

A skill segue o formato `SKILL.md` (Agent Skills). Em agentes que leem esse formato, copie a pasta
`plugins/revisar-mr/skills/revisar-mr` para a pasta de skills do agente:

```bash
git clone https://github.com/gui-mcastro/claude-skills.git
cp -r claude-skills/plugins/revisar-mr/skills/revisar-mr ~/.codex/skills/   # exemplo: Codex
```

Em agentes sem suporte a skills, aponte para o arquivo no `AGENTS.md` (ou equivalente) do projeto:

```markdown
Para revisar MR, siga `<caminho>/revisar-mr/SKILL.md` e as referências da pasta `references/`.
```

O modo GitLab da skill depende de um navegador controlado pelo agente (no Claude, o Claude in
Chrome) logado no GitLab do Sebrae. O modo local só precisa de `git`.
