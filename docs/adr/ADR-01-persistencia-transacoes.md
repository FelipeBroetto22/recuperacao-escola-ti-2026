# ADR-01 — Persistência em SQLite com SQL parametrizado e constraints

Status: Aceito

## Contexto
A API precisa manter parceiros, projetos, atividades, chaves de idempotência e eventos de webhook salvos após reinício (usando volume `/data` no container) e impedir duplicidades mesmo com chamadas simultâneas.

## Alternativas consideradas
1. **Dicionários em memória:** simples, mas perde tudo ao reiniciar e não atende o requisito de persistência.
2. **PostgreSQL + ORM (SQLAlchemy):** robusto para múltiplas instâncias, mas exige subir um segundo container/serviço e adiciona camadas extras de configuração.
3. **SQLite com `sqlite3` nativo e SQL parametrizado (`?`):** grava em arquivo único em `/data`, sem depender de servidor externo, deixando transações e constraints explícitas.

## Decisão
Usar SQLite via módulo padrão `sqlite3` com `PRAGMA foreign_keys = ON`, `PRAGMA journal_mode = WAL`, `PRAGMA busy_timeout = 5000` e transações `BEGIN IMMEDIATE` nas operações de escrita, garantindo unicidade direto no banco (`UNIQUE` e `PRIMARY KEY`).

## Consequências
Positivas: fácil de empacotar em container único, queries fáceis de ler/explicar e proteção real no disco contra corrida local. Negativas: SQLite concentra escritas em um único arquivo, então não escalaria para vários containers escrevendo em servidores separados.