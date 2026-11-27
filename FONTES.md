# FONTES.md — Declaração de Consultas

> [!IMPORTANT]
> **Consulta é permitida — conteúdo sem rastreabilidade, não.**
> Preencha este arquivo **antes do push final** se você usou qualquer fonte
> além do seu próprio conhecimento e do material deste repositório. Se não
> usou nada, declare isso nas seções abaixo (transparência também conta ponto
> de confiança na correção).[^transparencia]

## 1. Sites / documentação consultados

> [!NOTE]
> **Só contam linhas numeradas da tabela** (`| 1 | <URL> | ...`). Links em
> texto corrido — inclusive o exemplo logo abaixo — **não são contados**
> como fonte declarada.

| # | URL | O que foi consultado | Onde aparece no entregável |
| --- | --- | --- | --- |
| 1 | https://github.com/github/spec-kit | Estrutura de artefatos SDD (constitution, spec, plan e tasks) | `docs/tasks.md`, `docs/constitution.md` e `docs/pesquisa.md` (P01) |
| 2 | https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html | Comparação de níveis de uso de especificação viva no projeto | `docs/pesquisa.md` (P01) |
| 3 | https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow | Funcionamento de feature branches vs fluxos com develop | `docs/pesquisa.md` (P12) |
| 4 | https://docs.github.com/en/get-started/using-github/github-flow | Uso de branches curtas e PRs direto na main | `docs/pesquisa.md` (P12) e fluxo de PRs |
| 5 | https://www.rfc-editor.org/rfc/rfc9457 | Campos do formato `application/problem+json` para respostas de erro | `docs/constitution.md` e `docs/spec.md` |
| 6 | https://www.rfc-editor.org/rfc/rfc9110 | Semântica dos códigos HTTP (201, 202, 204, 400, 404, 409, 422, 429, 503) | `docs/spec.md` |
| 7 | https://viacep.com.br/ | Formato da URL de consulta de CEP e retorno `{"erro": true}` | `docs/spec.md` e `docs/plan.md` |
| 8 | https://docs.python.org/3.12/library/decimal.html | Arredondamento `ROUND_HALF_UP` sem ponto flutuante para percentuais | `docs/constitution.md` e `docs/spec.md` |
| 9 | https://docs.python.org/3.12/library/sqlite3.html | Conexão nativa com SQLite e queries parametrizadas com `?` | `docs/constitution.md` e `docs/plan.md` |
| 10 | https://www.sqlite.org/lang_createtable.html | Regras de `UNIQUE`, `PRIMARY KEY` composta e `FOREIGN KEY` no SQLite | `docs/plan.md` |
| 11 | https://www.sqlite.org/lang_transaction.html | Isolamento e transações no SQLite sob escritas concorrentes | `docs/plan.md` |
| 12 | https://martinfowler.com/articles/practical-test-pyramid.html | Divisão entre testes de unidade, integração e servidores locais simulados | `docs/tests.md` |
| 13 | https://docs.pytest.org/en/stable/how-to/tmp_path.html | Uso de diretórios temporários isolados por teste para o arquivo SQLite | `docs/tests.md` |

*(Ex.: `https://docs.oracle.com/...` → sintaxe de `Optional` → `plan.md` na
seção de decisões. Esse link é ILUSTRATIVO — fora de linha numerada não
conta. Se nenhum site foi consultado, escreva: **"Nenhum site consultado."**)*

## 2. Uso de IA — **somente como consulta**

> [!WARNING]
> Usar IA **como agente** (ela edita arquivos, executa comandos, roda testes no
> seu lugar) é **proibido** e zera a prova. Usar IA como consulta (perguntas,
> explicações de conceito, revisão pontual, trechos que você copiou e entende) é
> permitido **desde que**:

1. a conversa seja **compartilhada** (botão Share) e o link fique **público**
   (ou acessível ao professor);
2. o link seja registrado abaixo, indicando **onde** o conteúdo foi usado;
3. você seja capaz de **explicar qualquer trecho** que a IA produziu — na
   dúvida, o professor pede o link e pergunta sobre o código.[^plagio]

| # | Link público da conversa | Onde o conteúdo foi usado |
| --- | --- | --- |
| 1 | https://share.gemini.google/U2Lwm3EOPL96 | Geração e explicação da sintaxe do diagrama ER em Mermaid das 5 tabelas | `docs/plan.md` |

*(Se nenhuma IA foi utilizada, escreva: **"Nenhuma IA utilizada."**)*

## 3. Compromisso

Declaro que todo o conteúdo deste repositório que não é de minha autoria direta
está declarado acima, e que consigo explicar qualquer trecho entregue — tenha
ele vindo da minha cabeça, de um site ou de uma IA consultada.

**Nome / RA:** Felipe Broetto Araujo — 23167564-2

[^transparencia]: Este arquivo é, ele mesmo, um exemplo de markdown bem
    usado: *alert* para a regra crítica, tabelas para os registros e *footnote*
    para justificativas. O mesmo padrão vale para os `.md` que você entregar.
[^plagio]: Rubrica comum da disciplina: conteúdo de LLM não declarado
    configura plágio e zera a prova — o link público da conversa é o que
    transforma "copiou" em "consultou".