# ADR-02 — Testes com banco SQLite real temporário e servidores HTTP locais

Status: Aceito

## Contexto
A suíte precisa rodar 100% sem internet e provar que regras de negócio, constraints de banco, concorrência, ViaCEP e entregas de webhook funcionam de ponta a ponta.

## Alternativas consideradas
1. **Chamar ViaCEP real ou webhook externo na internet:** quebraria sem rede e violaria as regras da avaliação.
2. **Mockar todas as funções internas e o banco de dados:** testes rodam rápido, mas não pegam erros reais de SQL, constraints `UNIQUE`/`FOREIGN KEY` nem timeouts HTTP de verdade.
3. **Banco SQLite real em arquivo temporário (`tmp_path`) + servidor HTTP local (`http.server`) nos testes de integração:** testa o caminho completo da API sem sair da máquina.

## Decisão
Combinar testes unitários diretos para funções puras (`debug/` e cálculos) com testes de integração batendo nas rotas FastAPI sobre um SQLite temporário real e servidores HTTP locais simulando ViaCEP e o receptor do webhook (respostas `200`, `400`, `429`, `500` e atraso de timeout).

## Consequências
Positivas: pega erros reais de SQL, transações e rede local sem depender de internet. Negativas: subir servidor local e gravar em disco deixa os testes alguns milissegundos mais lentos que mocks puros em memória.