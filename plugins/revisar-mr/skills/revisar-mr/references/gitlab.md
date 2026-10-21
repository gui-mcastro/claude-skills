# GitLab pelo Chrome

- GitLab: `https://gitlab.sebrae.com.br`. Projetos: `df/cerebro` (API
  `/api/v4/projects/df%2Fcerebro/...`) e `df/migrations-sgi` (API
  `/api/v4/projects/df%2Fmigrations-sgi/...`).
- Chame a API com `fetch` numa aba do GitLab, já logada. Para POST/PUT, envie o header
  `X-CSRF-Token` lido de `meta[name=csrf-token]`. A mesma aba serve para os dois projetos.
- O navegador interno do app não tem login. Use o Claude in Chrome.
- O retorno do `javascript_tool` é truncado e às vezes bloqueado ("Cookie/query string data").
  Para ler muito texto, escreva o resultado num `<pre>` no `body` e leia com `get_page_text`, em
  partes de até ~45 mil caracteres. Devolva do `fetch` só `status` e `id`.
- Operações longas (diff de arquivo grande) travam o renderer por mais de 45 s. Rode em segundo
  plano (`promise.then(...)` gravando em `window._x`) e consulte depois.

## Ler o MR

- `GET /merge_requests/<id>`: estado, autor, branches, descrição, `diff_refs`.
- `GET /merge_requests/<id>/discussions?per_page=100`: comentários (ignore `system`, `caio`,
  `group_*`).
- `GET /merge_requests/<id>/changes?access_raw_diffs=true`: diff por arquivo. Em MR grande o `diff`
  vem vazio (colapsado). Nesse caso baixe `/df/cerebro/-/merge_requests/<id>.diff` (texto completo),
  separe por `^diff --git ` e use o pedaço de cada arquivo.
- Arquivo da branch: `GET /repository/files/<path urlencoded>/raw?ref=<branch>`.
- Busca no repositório: `GET /search?scope=blobs&search=<termo>&ref=<branch>`. O índice pode estar
  desatualizado. Confirme lendo o arquivo raw.

## MR com centenas de arquivos

O diff do GitLab usa o merge-base de quando a versão do MR foi criada (`diff_refs.base_sha`), e a
`develop` pode ter avançado depois. Muitos arquivos "alterados" são só ruído.

- Compare o blob do arquivo na branch e na `develop` atual (`/repository/files/<path>?ref=...`,
  campo `blob_id`). Arquivo idêntico não é mudança do MR.
- Arquivo diferente cujo último commit na `develop` é mais novo e de outra pessoa
  (`/repository/commits?ref_name=develop&path=<path>`) também é ruído.
- `GET /repository/compare?from=develop&to=<branch>&straight=true` dá a diferença real de
  árvore, mas inclui commits novos da `develop` invertidos. Filtre com os dois critérios acima.
- Só reclame de escopo depois dessa checagem.

## Comentar

- Thread na linha: `POST /merge_requests/<id>/discussions` com `body` e
  `position = {position_type: "text", base_sha, start_sha, head_sha, new_path, old_path, new_line}`
  (shas de `diff_refs`).
  - Linha adicionada: só `new_line`.
  - Linha removida: só `old_line`.
  - Linha de contexto: `old_line` **e** `new_line`.
- Calcule `new_line` contando as linhas do hunk (`@@ -a,b +c,d @@`) no diff do próprio MR, não no
  diff local. A linha precisa estar no diff do MR; se não estiver, use comentário geral.
- Confira depois: a nota criada deve ter `type: "DiffNote"`.
- Responder thread: `POST /merge_requests/<id>/discussions/<discussion_id>/notes`.
- Resolver thread própria: `PUT /merge_requests/<id>/discussions/<discussion_id>?resolved=true`.
- Comentário geral: mesmo POST de discussions, sem `position`.
