# ADR-03 — Idempotency-Key atômica no banco por rota e hash do payload

Status: Aceito

## Contexto
As rotas `POST /projetos`, `POST /projetos/{id}/atividades` e `POST /projetos/{id}/publicacao` exigem `Idempotency-Key` (8..100 caracteres) retida por no mínimo 24h, devolvendo a mesma resposta em replay e `409 idempotencia_conflito` se o payload mudar.

## Alternativas consideradas
1. **Cache em memória (dicionário Python):** perde as chaves ao reiniciar o processo e falha entre processos.
2. **Fazer `SELECT` antes e `INSERT` no fim sem transação travada:** sofre de corrida (*check-then-act*) quando duas requisições idênticas chegam juntas.
3. **Tabela `idempotencia` com `PRIMARY KEY (rota, chave)` e reserva em transação `BEGIN IMMEDIATE`:** o próprio banco impede duas gravações iguais ao mesmo tempo.

## Decisão
Calcular o hash SHA-256 do JSON serializado com chaves ordenadas (`json.dumps(..., sort_keys=True, separators=(",", ":"))`) e gravar `(rota, chave)` dentro de transação `BEGIN IMMEDIATE` (com `expira_em` de +24h). Validações de formato (JSON quebrado `400` ou campos inválidos `422`) acontecem antes e não consomem a chave.

## Consequências
Positivas: garante escopo separado por rota, sobrevive a reinício e evita registros duplicados em chamadas concorrentes. Negativas: exige serializar e calcular hash do corpo em toda criação protegida.