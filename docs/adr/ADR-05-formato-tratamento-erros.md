# ADR-05 — Padronização de erros com Problem Details (RFC 9457) e Correlation ID

Status: Aceito

## Contexto
Respostas de erro soltas (como `{"detail": "..."}` padrão do framework) dificultam saber qual regra de negócio falhou e rastrear um erro específico nos logs sem expor detalhes internos do banco.

## Alternativas consideradas
1. **Usar o erro padrão do FastAPI:** mistura formatos diferentes entre erro `422` de validação e erros manuais, sem `code` padronizado.
2. **Envelope próprio simples (`{"erro": "...", "codigo": "..."}`):** não segue o padrão HTTP `application/problem+json` pedido no contrato.
3. **Handlers globais gerando `application/problem+json` (RFC 9457) + middleware de `X-Correlation-ID`:** centraliza todos os erros num único formato.

## Decisão
Interceptar erros de negócio, validação, JSON inválido e falhas inesperadas em handlers únicos do FastAPI, devolvendo sempre `Content-Type: application/problem+json` com `type`, `title`, `status`, `detail`, `instance`, `code` e `correlation_id` (além da lista `errors` com `field` e `message` em falhas `422`). Um middleware gera ou reaproveita `X-Correlation-ID` em toda requisição, devolve no header e usa nos logs sem incluir SQL, stack trace, segredos ou endereço completo.

## Consequências
Positivas: contrato uniforme em qualquer falha e fácil correlação entre resposta do cliente e linha de log. Negativas: exige mapear também os erros internos do FastAPI/Pydantic para o mesmo envelope.