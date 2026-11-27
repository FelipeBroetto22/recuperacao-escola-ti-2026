# ADR-04 — Outbox persistente e política de retentativas (Webhook e ViaCEP)

Status: Aceito

## Contexto
Integrações externas falham por rede, timeout ou erro temporário. Na publicação de projeto (`202`), não podemos perder o evento se o receptor cair nem repetir erros permanentes (`400`), respeitando o limite `TENTATIVAS_WEBHOOK=3` da variante.

## Alternativas consideradas
1. **Enviar o webhook de forma síncrona dentro do `POST /publicacao` com `sleep`:** trava a resposta da API e perde o envio se o processo reiniciar no meio.
2. **Repetir qualquer erro HTTP com intervalo fixo:** desperdiça chamadas em erros `400`/`404` e gera pico sincronizado de tráfego (*retry storm*).
3. **Gravar na tabela `outbox_eventos` e processar via worker com retry seletivo, backoff exponencial + jitter e `Retry-After`:** separa o `202` da entrega e sobrevive a reinício.

## Decisão
No webhook, gravar o evento como `pendente` na outbox e processar por comando/worker com timeout explícito, repetindo até `3` tentativas totais apenas em falha de rede/timeout, `429` (respeitando `Retry-After` limitado a um teto) e `5xx`, usando atraso base exponencial com pequeno sorteio (*jitter*); demais `4xx` marcam `falhou` na hora. Na ViaCEP (chamada síncrona no cadastro), limitar a no máximo 1 nova tentativa rápida apenas em rede/timeout/`5xx` para não segurar a requisição do usuário.

## Consequências
Positivas: respostas rápidas no `202`, zero perda em reinício e sem insistir em erro de cliente. Negativas: exige rodar o processador da outbox para efetivar o envio.