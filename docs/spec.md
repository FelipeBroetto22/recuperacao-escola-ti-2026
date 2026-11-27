# Especificação Funcional e Regras — Conecta Comunidade

Parâmetros ativos desta variante (`variante/params.json`): `PORTA_API=8109`, `LIMITE_PAGINACAO=10`, `HORAS_LIMITE_ATIVIDADE=8` (`800` centésimos), `TENTATIVAS_WEBHOOK=3`.

## 1. Convenções Gerais

- Respostas JSON usam `application/json` e erros usam `application/problem+json` com os campos obrigatórios `type`, `title`, `status`, `detail`, `instance`, `code` e `correlation_id`.
- Toda resposta HTTP (sucesso ou erro) devolve o header `X-Correlation-ID` (recebido na requisição ou gerado pela API).
- Datas/horas usam ISO-8601 com fuso explícito e são salvas em UTC.
- Paginação usa `pagina >= 1` (padrão `1`) e `tamanho` de `1..50` (padrão `10`), respondendo `{ "dados": [...], "paginacao": { "pagina", "tamanho", "total", "total_paginas" } }`.

## 2. Requisitos Funcionais e Critérios de Aceite

- `RF-01` (Saúde): `GET /saude` retorna `200` com `{"status": "ok", "versao": "1.0.0"}` sem chamar ViaCEP ou webhook.
- `RF-02` (Criar e consultar parceiros): `POST /parceiros` valida dados (`nome` 3..100 sem só espaços, `email` único em minúsculas, `cep` com 8 dígitos, `complemento` opcional até 120), consulta ViaCEP e retorna `201`. `GET /parceiros` pagina por `id` crescente com filtros `nome` (parcial, sem diferenciar maiúsculas) e `ativo` (`true`/`false`). `GET /parceiros/{id}` retorna `200` ou `404 parceiro_nao_encontrado`.
- `RF-03` (Editar, ativar/desativar e excluir parceiro): `PUT /parceiros/{id}` exige `nome`, `email`, `cep` e `complemento` (refazendo busca na ViaCEP se o CEP mudar) e retorna `200`. `PATCH /parceiros/{id}` aceita apenas `{"ativo": true|false}`. `DELETE /parceiros/{id}` retorna `204` se não houver projetos vinculados.
- `RF-04` (Criar e consultar projetos): `POST /projetos` exige header `Idempotency-Key` (8..100 caracteres), valida `titulo` (3..120), `descricao` (10..2000), `parceiro_id` ativo e `horas_previstas_centesimos` (100..1000000), criando em `rascunho` (`201`). `GET /projetos` filtra por `status` e `parceiro_id`. `GET /projetos/{id}` retorna `200` ou `404 projeto_nao_encontrado`.
- `RF-05` (Editar, mudar status e excluir projeto): `PUT /projetos/{id}` edita apenas em `rascunho`. `PATCH /projetos/{id}/status` aplica transições válidas (`rascunho -> ativo -> encerrado`). `DELETE /projetos/{id}` retorna `204` apenas em `rascunho` sem atividades.
- `RF-06` (Criar e consultar atividades): `POST /projetos/{id}/atividades` exige `Idempotency-Key`, valida `titulo` (3..120), `inicio` (ISO-8601 com offset), `voluntario` (3..100) e `horas_previstas_centesimos` (múltiplo de 25, entre `25` e `800`), criando em `planejada` (`201`). `GET /atividades` filtra por `projeto_id`, `status`, `de` e `ate` (intervalo inclusivo em `inicio`), ordenando por `inicio` crescente e `id` crescente. `GET /atividades/{id}` retorna `200` ou `404 atividade_nao_encontrada`.
- `RF-07` (Editar, finalizar e excluir atividade): `PUT /atividades/{id}` e `DELETE /atividades/{id}` (`204`) funcionam apenas em `planejada`. `PATCH /atividades/{id}/status` muda para `concluida` (exigindo `horas_realizadas_centesimos` múltiplo de 25 entre `25` e `800`) ou `cancelada`.
- `RF-08` (Relatório do projeto): `GET /projetos/{id}/relatorio` retorna contagens por status, soma de `horas_previstas_centesimos` (`planejada` + `concluida`), soma de `horas_realizadas_centesimos` (`concluida`) e `percentual_conclusao` como string de duas casas arredondada `half-up` (`"0.00"` quando previstas for zero).
- `RF-09` (Publicação na outbox): `POST /projetos/{id}/publicacao` exige `Idempotency-Key`, configuração de webhook válida e projeto `ativo`, gravando evento `pendente` na outbox e respondendo `202` com `evento_id`, `projeto_id` e `status`.
- `RF-10` (Entrega de webhook e consulta): Worker envia `POST` para `WEBHOOK_URL` com headers `X-Event-ID`, `X-Idempotency-Key`, `X-Webhook-Timestamp` e `X-Webhook-Signature` (`sha256=<hmac>`), repetindo até `3` tentativas totais em falha de rede/timeout, `429` ou `5xx`. `GET /integracoes/eventos/{evento_id}` retorna `200` com estado e tentativas sem expor segredos.

## 3. Requisitos Não Funcionais

- `RNF-01`: Erros padronizados pela RFC 9457 + `X-Correlation-ID` em todas as respostas.
- `RNF-02`: Banco SQLite persistido em volume (`/data`) com chaves estrangeiras e constraints `UNIQUE` ativas.
- `RNF-03`: Timeouts explícitos em chamadas externas (`HTTP_TIMEOUT_MS` e `WEBHOOK_TIMEOUT_MS`), máximo de 1 retry na ViaCEP e backoff exponencial com jitter + `Retry-After` no webhook.
- `RNF-04`: Chave de idempotência isolada por rota, comparada por hash SHA-256 do JSON canônico, retida por pelo menos 24h e protegida contra chamadas simultâneas no banco.
- `RNF-05`: SQL parametrizado e sanitização de erros/logs sem vazar stack trace, SQL, segredo ou endereço completo.
- `RNF-06`: Container ouvindo na porta `8080`, rodando sem usuário root, com `.dockerignore` e `HEALTHCHECK` em `/saude`.
- `RNF-07`: Pelo menos 30 testes automatizados em `tests/app/` rodando 100% sem internet e 3 ciclos de TDD registrados.

## 4. Regras de Negócio

### Parceiros
- `RN-P01`: E-mail repetido retorna `409` (`email_em_uso`), inclusive em requisições concorrentes.
- `RN-P02`: CEP inexistente (`erro: true`) ou HTTP 400 na ViaCEP retorna `422` (`cep_invalido`).
- `RN-P03`: Timeout, falha de conexão, JSON inválido ou 5xx na ViaCEP retorna `503` (`viacep_indisponivel`).
- `RN-P04`: Excluir parceiro que possui qualquer projeto retorna `409` (`parceiro_com_projetos`).
- `RN-P05`: Desativar parceiro com projeto `ativo` retorna `409` (`parceiro_com_projetos_ativos`).
- `RN-P06`: Projetos em `rascunho` ou `encerrado` não bloqueiam a desativação do parceiro.
- `RN-P07`: Se a consulta de CEP falhar na criação ou no `PUT`, nada é salvo nem alterado parcialmente.

### Projetos
- `RN-J01`: Criar projeto para parceiro inexistente retorna `404` (`parceiro_nao_encontrado`).
- `RN-J02`: Parceiro inativo não pode receber projeto novo nem ter projeto ativado, retornando `409` (`parceiro_inativo`).
- `RN-J03`: Só valem as mudanças `rascunho -> ativo -> encerrado`; outras retornam `409` (`transicao_status_invalida`).
- `RN-J04`: Ativar projeto exige parceiro ativo e ao menos 1 atividade `planejada`, senão `409` (`projeto_sem_atividade_planejada`).
- `RN-J05`: Encerrar projeto exige zero atividades `planejada`, senão `409` (`projeto_com_atividades_planejadas`).
- `RN-J06`: `PUT` ou `DELETE` em projeto `ativo` ou `encerrado` retorna `409` (`projeto_nao_editavel`).
- `RN-J07`: `DELETE` em projeto `rascunho` que já tem atividades retorna `409` (`projeto_com_atividades`).
- `RN-J08`: Projeto `encerrado` não aceita edição, novas atividades nem publicação (`409`).

### Atividades
- `RN-A01`: Só projetos `rascunho` ou `ativo` aceitam novas atividades (em `encerrado` retorna `409 projeto_nao_editavel`).
- `RN-A02`: Concluir exige `status="concluida"` junto com `horas_realizadas_centesimos` válido (`422 validacao_falhou` se faltar).
- `RN-A03`: Cancelar usa `status="cancelada"` e não pede horas realizadas (se enviadas no cancelamento, rejeita com `422 validacao_falhou`).
- `RN-A04`: Só vale `planejada -> concluida` ou `planejada -> cancelada`; repetir ou trocar estado final retorna `409` (`atividade_finalizada`).
- `RN-A05`: `PUT` ou `DELETE` em atividade `concluida` ou `cancelada` retorna `409` (`atividade_finalizada`).
- `RN-A06`: Atividades `cancelada` ficam fora da soma de previstas e realizadas no relatório.
- `RN-A07`: Horas previstas e realizadas são múltiplos de 25 no intervalo `25..800` (`422 validacao_falhou` fora disso).

## 5. Casos de Uso Principais

- `UC-01` (Cadastrar parceiro com CEP): Recebe dados -> valida formato -> consulta ViaCEP -> grava parceiro no SQLite (`201`) ou aborta sem gravar (`422`/`503`).
- `UC-02` (Planejar e ativar projeto): Cria projeto em `rascunho` com `Idempotency-Key` -> cadastra atividade `planejada` -> ativa projeto via `PATCH /projetos/{id}/status`.
- `UC-03` (Executar atividades e tirar relatório): Conclui ou cancela atividades -> consulta `GET /projetos/{id}/relatorio` com somas inteiras e percentual `Decimal`.
- `UC-04` (Encerrar projeto e desativar parceiro): Finaliza todas as atividades pendentes -> muda projeto para `encerrado` -> permite desativar o parceiro sem projetos ativos.
- `UC-05` (Publicar projeto na outbox): Em projeto `ativo`, grava evento `projeto.publicado` com idempotência (`202`) -> worker assina payload com HMAC e entrega ao receptor HTTP.

## 6. Decisões sobre Pontos Abertos e Precedência

1. **Ordem de validação de erros:** (1) JSON malformado ou query param inválido -> `400`; (2) `Idempotency-Key` ausente/inválida ou campos do body fora do schema -> `422`; (3) conflito de payload na mesma `Idempotency-Key` -> `409 idempotencia_conflito`; (4) recurso pai/alvo inexistente -> `404`; (5) regras de estado e negócio -> `409` ou `503`.
2. **Fronteira da idempotência:** Requisições barradas antes da lógica de negócio por JSON quebrado (`400`) ou chave/body inválidos (`422`) não gravam resposta na tabela de idempotência; a gravação vale para a execução da operação válida.
3. **`PUT /parceiros/{id}` com mesmo CEP:** Quando o CEP enviado no `PUT` é idêntico ao CEP já salvo no parceiro, o endereço atual é mantido sem chamar a ViaCEP novamente; se o CEP mudar, a consulta é refeita antes de salvar.
4. **Publicação de projeto fora de `ativo`:** Tentar publicar projeto em `rascunho` ou `encerrado` retorna `409` (`projeto_nao_publicavel`).