# Regras de revisão

Valem nos dois modos (GitLab e Local). No modo Local, "comentar" significa "listar no relatório".

## Regras gerais

1. **MR já mergeado** (modo GitLab): se o estado for `merged`, não revise. Só informe.

2. **Comentários anteriores** (modo GitLab): antes de revisar, leia as discussões já existentes no MR.
   - Ignore completamente os comentários do usuário `caio` e de bots `group_*`. São automáticos
     (SonarQube e parecer de IA).
   - Para cada apontamento de outra pessoa, confira no diff atual se o autor corrigiu. Se não
     corrigiu, responda na própria thread dizendo o que ainda falta.
   - Não repita na sua revisão pontos já levantados por outra pessoa.

3. **Comentários na linha**: cada problema vira uma thread separada, ancorada na linha do diff que
   tem o erro.
   - Quando o problema ocupa várias linhas (método, bloco, array, statement SQL), ancore na
     **última linha** do trecho. Ex.: conversão de data repetida em `'options' => [...]` → ancore no
     `],`; FK faltando num `CREATE TABLE` → ancore no `);`.
   - Problema de uma linha só: ancore nela mesma.
   - Comentário geral só para o que não tem linha no diff (ex.: arquivo fora do diff que quebra por
     causa da mudança).

4. **Nunca aprove**: não chame `/approve` em hipótese alguma. Se não houver problemas e os
   apontamentos anteriores estiverem corrigidos, informe que o MR está pronto para aprovação.

5. **Exceção conhecida**: `save(false)` em `Fornecedor` (ex.: `User::salvarSolicitacaoEvento`) é
   intencional. Não aponte.

6. **Formatação não reprova**: não comente indentação, espaços, linhas em branco, vírgula final,
   quebra de linha nem ordem de `use`. Se só houver isso, trate como sem problemas.

7. **Repetição de código**: verifique duplicação dentro do diff (blocos copiados entre
   arquivos/métodos) e em relação ao projeto (helpers do `BaseModel` como `dropDownList` e
   `dropDownListRelation`, métodos e relações de outros models, componentes `Yii::$app->util`).
   Busque no repositório antes de apontar e indique o método/arquivo existente a reutilizar.

8. **Quebra fora do diff**: quando o MR apaga ou renomeia classe, método, relação, view ou ação,
   procure os usos no repositório inteiro (inclusive módulos como `sacCliente`, `fornecedor` e
   `aquisicao`). Uso remanescente é erro e vira comentário geral com arquivo e linha.

9. **MR de migrations**: o banco fica no projeto `df/migrations-sgi`. Procure o link na descrição e
   nos comentários do autor.
   - Revise junto com o código: tabelas, colunas, tipos e FKs devem bater com o model, as rules e
     as views (nomes de colunas, nullable × `required`, tamanho × `max`). Divergência é erro.
   - Aplique também as regras 2, 3, 4 e 6 no MR de migrations.
   - Apontamento sobre banco vai **no MR de migrations**, na linha. Não repita no Cérebro.
   - No MR do Cérebro, um único comentário geral: `Olhar migrations, $n comentários` + link do MR de
     migrations, com `$n` = threads/respostas criadas nele nesta execução. Se `$n` = 0, não comente.
   - Se o Cérebro usa tabela/coluna nova (atributo novo nas `rules`/`@property`, `tableName()` novo,
     coluna renomeada) e não há link de migrations, comente pedindo o link. Antes, procure em
     `df/migrations-sgi` um MR já existente e cite-o.

### Revisão das migrations (procedures, functions, views, inserts, updates, DDL)

- **Objetivo**: integridade, performance e segurança; pegar falhas de lógica, vulnerabilidades e
  gargalos antes de produção. Feedback técnico, preciso e acionável.
- **Funcionalidade e lógica**: resolve o problema? Cobre erro e borda (NULL, registro inexistente,
  duplicidade, reexecução)?
- **Desempenho**: filtros/joins em colunas indexadas, sem função na coluna do filtro, sem
  `SELECT *`, set-based em vez de loop, transações curtas.
- **Segurança**: SQL dinâmico com `sp_executesql` e parâmetros, nunca concatenação de entrada. Nada
  de CPF, senha ou token em log.
- **Manutenibilidade**: nomes claros e no padrão do banco; comentário explicando lógica não óbvia.
- **Integridade**: PK/FK definidas, unicidade no banco quando o model valida `unique`, `down()`
  desfazendo tudo o que o `up()` cria. **Não cobre `BEGIN TRY`/`BEGIN TRAN` no `up()`/`down()`.**
  Transação só é exigida em procedures/functions/scripts com várias escritas relacionadas.
- **Pasta do mês**: migration nova em `database/migrations/{ano}/{mes}/` do mês e ano **atuais**.
  Não valide o nome do arquivo. Arquivos fora da pasta: **uma única thread geral** no formato abaixo
  (se já existir thread aberta sobre isso, responda nela). Conta como 1 comentário.

  ```
  Mover para `database/migrations/<ano>/<mes>/`:
  <ul>
  <li>2026_09_11_112738_alter_git_parametro_cadastro_table.php</li>
  </ul>
  ```
- **Tipos de texto**: `nvarchar` em vez de `varchar`. Exceção: formato fixo só com dígitos/ASCII
  (CPF, CNPJ, CEP, telefone) pode ser `varchar`/`char`.
- **Nome de FK**: `FK_{tabela_origem}_{campo_origem}` (ex.:
  `FK_git_usuario_web_service_suborigem_cd_suborigem`).
- **Nome de índice**: `IDX_{tabela_origem}__{campo_origem}`, com dois underscores (ex.:
  `IDX_git_usuario_web_service_suborigem__cd_suborigem`).
- **SQL bruto é aceito e preferido**. Não critique ausência de Query Builder.
- **Legado (loops/cursores)**: se o loop chama outras procedures/rotinas, não reprove; só sugira
  set-based como melhoria ("sugestão, não bloqueia"). Se não chama, reprove e peça set-based.

## Padrões Yii2

Aponte cada violação com a regra, o porquê, de forma clara e objetiva, e a correção sugerida, com
exemplo curto quando ajudar. Seja direto.

- **Yii 1 — Fat Model / Skinny Controller**: proíba consultas, filtros complexos e manipulação de
  dados na Controller e na View. Acesso a dados fica em Models ou Repositories.
- **Yii 2 — Nome de relacionamentos**: todo `hasOne`/`hasMany` começa com `getRel` (não `getUser`,
  `getColaboradorRel`). O padrão é `getRel` + nome da classe relacionada, mas o que vem depois do
  `getRel` pode ter a ver com **o nome da classe** ou com **o campo da relação** (a coluna do
  model atual no `hasOne`/`hasMany`, ex.: `cd_responsavel` em `['id' => 'cd_responsavel']`):
  - **Passa pela classe**: palavras inteiras do nome da classe (divisão PascalCase), na mesma
    ordem, com singular/plural livre. Ex.: classe `UsuarioWebServiceSuborigem` →
    `getRelUsuarioWebServiceSuborigem`, `getRelSuborigem` e `getRelSuborigens` passam; classe
    `SistemaSuborigem` → `getRelSuborigem` passa.
  - **Passa pelo campo**: palavras do nome do campo sem o prefixo (`cd_`, `id_`, `nr_`, `tp_`,
    `st_` etc.), na mesma ordem, aceitando a abreviação do campo escrita por extenso. Ex.:
    `hasOne(User::class, ['id' => 'cd_responsavel'])` → `getRelResponsavel`;
    `['id' => 'id_criacao']` → `getRelCriacao`; `['id' => 'cd_usu_resp']` → `getRelUsuResp` ou
    `getRelUsuarioResponsavel`; `['cd_unidade' => 'cd_unidade_resp']` → `getRelUnidadeResp`.
  - **Não passa (muito diferente)**: palavra que não está nem na classe nem no campo
    (`getRelItens`, `getRelDados`, `getRelLista`), abreviação que não vem do campo (`getRelUwss`),
    tradução ou sinônimo (`getRelSubOrigins`, `getRelGestor` para `cd_responsavel`), ou o nome
    sem o `getRel` (`getUser`). Nesses casos, peça `getRel` + **nome exato da classe** (ou do
    campo, se houver mais de uma relação para a mesma classe).
  - Se duas relações do mesmo arquivo ficarem ambíguas com o nome parcial (ex.: `getRelSuborigem`
    para `SistemaSuborigem` e para `UsuarioWebServiceSuborigem`), peça o nome exato da classe.
  - **Exceção**: aceite outro nome (ex.: `getRelCriacao`, `getRelResponsavel` para `User`) só se
    **já existir** no mesmo arquivo (incluindo a classe base/ActiveQuery, como
    `FornecedorActiveQuery`) um `getRel<NomeDaModel>` para essa model. Confira no arquivo da branch
    de origem antes de apontar.
  - Se não existir, peça para renomear e atualizar os usos (`->rel...`, `with()`/`joinWith()`,
    `@property`).
- **Yii 3 — Use a relação, não consulta manual**: proíba `getCampo`, `getField` ou SQL direta para
  ler atributos de entidade já relacionada. Ex.: em vez de
  `User::getCampo('email', ['id' => $this->cd_colaborador])`, use `$this->relColaborador->email`.
- **Yii 4 — Sem JS/CSS inline em views**: JS em arquivo externo com `registerJsFile` e
  `'appendTimestamp' => true`; CSS com `registerCssFile`. Aponte `<script>`/`<style>` inline e
  `registerJsFile` sem `appendTimestamp` (grafia exata).
- **Yii 5 — IDs de Panel**: `idPanel` sempre começa com `clp-` e segue em kebab-case (minúsculo, sem
  acento, espaços e símbolos viram hífen). O padrão é `clp-` + `'title'` inteiro, mas o que vem
  depois do `clp-` pode ser **parte do título**:
  - **Passa**: formado por palavras inteiras do título, na mesma ordem, podendo omitir palavras
    (inclusive preposições/artigos como `de`, `da`, `do`), com singular/plural livre. Ex.:
    `'Avaliações Antes de 2026'` → `clp-avaliacoes-antes-de-2026`, `clp-avaliacoes-2026` e
    `clp-avaliacoes` passam; `'Origem - Suborigem'` → `clp-origem-suborigem` e `clp-suborigem`
    passam; `'Relação de Colaboradores'` → `clp-relacao-colaboradores` e `clp-colaboradores` passam.
  - **Não passa (muito diferente)**: sem o prefixo `clp-`; fora de kebab-case (`clp_dados`,
    `clpDados`, acento); palavra que não está no título (`clp-dados` para `'Origem - Suborigem'`,
    `clp-form-usuario-web-service-origem` para `'Origem - Suborigem'`); abreviação (`clp-os`);
    tradução. Nesses casos, peça `clp-` + o título inteiro em kebab-case.
  - Se dois panels da mesma tela ficarem com o mesmo id parcial, peça o título inteiro.
  - Título dinâmico (variável/concatenação): exija só `clp-` + kebab-case descritivo.
  - Ao pedir renomeação, lembre de conferir JS/CSS que usam o id antigo.
- **Yii 6 — Name/id de campos com model binding**: `name='Model[atributo]'` e
  `id='model-atributo'` (model em minúsculo, atributo preservado). Valide input, select, Select2,
  DepDrop e dynamic grids. Explique a relação com `load()`/`save()` e os conflitos de DOM.
