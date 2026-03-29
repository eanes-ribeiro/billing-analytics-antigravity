# Padrões dbt para Faturamento (Billing Patterns)

## 1. Estratégia Incremental Idempotente
Ao processar fechamentos de faturamento, NUNCA utilize `append` cego.
- **Padrão:** Use `materialized='incremental'` com `unique_key` composta.
- **Implementação:** A `unique_key` deve ser sempre `['contract_id', 'reference_month']`.
- **Comportamento:** Se o job rodar duas vezes para a mesma janela temporal, ele fará um `merge` (upsert), atualizando os valores em vez de duplicar as faturas.

## 2. Rastreamento de Histórico (SCD Tipo 2)
Cálculos de pro-rata dependem de saber exatamente quando um plano mudou de preço.
- **Padrão:** Use snapshots do dbt para a tabela de `dim_contract`.
- **Implementação:** Filtros em `int_billing` devem sempre cruzar a data do evento (ex: `usage_date`) garantindo que esteja entre `dbt_valid_from` e `dbt_valid_to`.

## 3. Testes Financeiros (Generic & Singular Tests)
- **Primary Key:** Toda tabela Mart de faturamento deve ter teste de `unique` e `not_null`.
- **Monetary Types:** Nenhum modelo pode expor campos monetários sem cast para `DECIMAL(18,2)` ou `(18,4)`.
- **Custom Rule:** Crie testes singulares para garantir que `desconto_total <= valor_bruto` (evitando faturas negativas).