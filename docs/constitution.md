# Constituição do Projeto

Regras fixas que valem para todo o desenvolvimento da API Conecta Comunidade:

1. A especificação vem antes da implementação. Todo requisito recebe um código (`RF`, `RNF`, `RN`, `UC`, `CT`, `ADR`) repetido nos testes e PRs.
2. Nos ciclos de TDD, o teste precisa falhar primeiro antes do código que faz ele passar.
3. Nenhum teste automatizado acessa a internet; ViaCEP e webhook usam servidores locais simulados.
4. Horas usam inteiros em centésimos (`100` = 1h) e percentuais usam `Decimal` com arredondamento `half-up`, nunca `float`.
5. Todo erro responde no formato `application/problem+json` (RFC 9457) e toda resposta inclui o header `X-Correlation-ID`.
6. Unicidade de e-mail e chaves de idempotência são garantidas por constraints e transações no banco SQLite, não apenas em memória.
7. Todas as consultas SQL usam parâmetros (`?`), e logs nunca exibem segredos, assinaturas completas, SQL, stack traces ou endereços inteiros.
8. Todo trabalho entra via branch e Pull Request ligado a uma issue, mantendo intactos os caminhos protegidos do repositório.
9. Toda documentação ou conversa consultada é declarada em `FONTES.md`.
10. Exemplos, testes e massas de dados usam apenas nomes, e-mails e organizações fictícios.