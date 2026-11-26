# Planejamento de Tarefas — Conecta Comunidade

Divisão do trabalho da semana ligando os requisitos do enunciado As issues e entregas.

## Parâmetros da variante (`variante/params.json`)

- `PORTA_API`: `8109` (porta interna no container: `8080`)
- `LIMITE_PAGINACAO`: `10`
- `HORAS_LIMITE_ATIVIDADE`: `8` (`800` centésimos de hora na API)
- `TENTATIVAS_WEBHOOK`: `3`

## Mapa resumido de códigos de requisitos

### Requisitos Funcionais (`RF`)
- `RF-01`: Saúde da aplicação (`GET /saude`)
- `RF-02`: Criação e consulta de parceiros com busca de CEP (`POST /parceiros`, `GET /parceiros`, `GET /parceiros/{id}`)
- `RF-03`: Atualização, ativação/desativação e remoção de parceiros (`PUT`, `PATCH`, `DELETE /parceiros/{id}`)
- `RF-04`: Criação com `Idempotency-Key` e listagem/detalhe de projetos (`POST /projetos`, `GET /projetos`, `GET /projetos/{id}`)
- `RF-05`: Edição em rascunho, mudança de status e exclusão de projetos (`PUT`, `PATCH /status`, `DELETE /projetos/{id}`)
- `RF-06`: Criação com `Idempotency-Key` e consulta de atividades com filtro de período (`POST /projetos/{id}/atividades`, `GET /atividades`, `GET /atividades/{id}`)
- `RF-07`: Edição, conclusão/cancelamento e exclusão de atividades (`PUT`, `PATCH /status`, `DELETE /atividades/{id}`)
- `RF-08`: Relatório consolidado do projeto em centésimos de hora e percentual `half-up` (`GET /projetos/{id}/relatorio`)
- `RF-09`: Enfileiramento de publicação de projeto ativo na outbox (`POST /projetos/{id}/publicacao`)
- `RF-10`: Worker de entrega de webhook assinado (`HMAC-SHA256`) e consulta de evento (`GET /integracoes/eventos/{evento_id}`)

### Requisitos Não Funcionais (`RNF`)
- `RNF-01`: Erros no formato `application/problem+json` (RFC 9457) e `X-Correlation-ID` em todas as respostas
- `RNF-02`: Persistência SQLite em `/data` preservada após reinício e constraints no banco
- `RNF-03`: Timeouts explícitos, retry seletivo, backoff exponencial com jitter e respeito a `Retry-After`
- `RNF-04`: Idempotência atômica no banco (retenção mínima de 24h) e proteção contra corrida/concorrência
- `RNF-05`: Queries parametrizadas e logs sem dados sensíveis (sem segredo, SQL, stack trace ou endereço completo)
- `RNF-06`: `Containerfile` na porta `8080`, usuário não-root, `HEALTHCHECK` e `.dockerignore`
- `RNF-07`: Testes automatizados locais sem uso de internet (mínimo de 30 testes e 3 ciclos TDD)

## Backlog por Issue e Fatia de Entrega

| Issue | Escopo da tarefa | Requisitos / Regras / Pesquisa | Entregáveis principais | Status |
| --- | --- | --- | --- | --- |
| `#2` | Planejamento inicial e primeiras pesquisas | `P01`, `P12` | `docs/tasks.md`, base de `docs/pesquisa.md` e `FONTES.md` | Em andamento |
| `#3` | Especificação SDD, OpenAPI 3.1 e ADRs | `RF-01..10`, `RNF-01..07`, `P02..P05`, `P07..P11`, `P14` | `constitution.md`, `spec.md`, `plan.md`, `tests.md`, `openapi.yaml`, `ADR-01..05` | Pendente |
| `#4` | Diagnóstico e correção dos 4 casos de debugging | `D01`, `D02`, `D03`, `D04`, `P05`, `P06` | `debug/` (código + testes de regressão) e `debug/diagnostico.md` | Pendente |
| `#5` | Base HTTP/SQLite + CRUD Parceiros + ViaCEP (TDD 1) | `RF-01..03`, `RN-P01..07`, `RNF-01..03`, `RNF-05` | `src/`, `tests/app/test_parceiros.py`, `evidencias/tdd/ciclo-01.md` | Pendente |
| `#6` | CRUD Projetos + `Idempotency-Key` (TDD 2) | `RF-04..05`, `RN-J01..08`, `RNF-04` | Rotas/regras de projetos, idempotência atômica, `evidencias/tdd/ciclo-02.md` | Pendente |
| `#7` | CRUD Atividades + Relatório do projeto (TDD 3) | `RF-06..08`, `RN-A01..07`, `RNF-04` | Rotas de atividades, relatório `half-up`, concorrência, `evidencias/tdd/ciclo-03.md` | Pendente |
| `#8` | Outbox, Webhook com HMAC e retentativas | `RF-09..10`, `RNF-03..05`, `P08`, `P10` | Publicação `202`, worker de webhook, `GET /integracoes/eventos/{id}`, testes com mock | Pendente |
| `#9` | Container, revisão ponta a ponta e documentação final | `RNF-06..07`, `P13` | `Containerfile`, `.dockerignore`, `.env.example`, `README.md`, `checklist-final.md` | Pendente |