# Pesquisa Aplicada

## P01 — SDD e rastreabilidade

Como não esquecer nenhuma regra de negócio na hora de programar e testar a API (são 3 recursos e 22 regras no enunciado).
- Fontes (2026-11-25): https://github.com/github/spec-kit e https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
- Aprendizado: o Spec Kit divide o projeto em `constitution`, `spec`, `plan` e `tasks` antes do código, enquanto o artigo no site do Martin Fowler mostra que a especificação perde valor se ficar abandonada depois da primeira entrega.
- Decisão: usar códigos curtos (`RF-xx`, `RNF-xx`, `RN-xx`, `CT-xx`) já em [docs/tasks.md](tasks.md) e repetir nos testes e PRs, em vez de deixar requisitos só em texto solto.
- Limitação: a conferência desses códigos entre documentos e testes precisa ser feita manualmente a cada PR.

## P12 — GitHub Flow, pull requests e revisão

Como organizar branches e PRs trabalhando sozinho em uma semana e sem mexer na pasta `.github/`.
- Fontes (2026-11-25): https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow e https://docs.github.com/en/get-started/using-github/github-flow
- Aprendizado: o guia da Atlassian mostra que fluxos maiores (como ter branch `develop` separada da `main`) servem para ciclos longos de release, enquanto o GitHub Flow usa branches curtas indo direto para a `main` via PR.
- Decisão: abrir uma branch curta por issue (`docs/issue-2-planejamento`), revisar o próprio diff no PR com `Closes #2` e descartar o uso de branch `develop` intermediária para não duplicar merges sem necessidade.
- Limitação: como não podemos criar workflows novos em `.github/`, preciso rodar o `pytest` no meu PC antes de abrir cada PR e usar o `/auto-correcao` que já existe na issue `#1`.