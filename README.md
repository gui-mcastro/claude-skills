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
