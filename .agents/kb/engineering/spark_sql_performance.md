
# Otimização Spark SQL para Faturamento

## 1. Partition Pruning (Filtragem Estática)
Motores de faturamento processam bilhões de linhas de TPV. O Catalyst Optimizer do Spark precisa eliminar arquivos no S3/GCS rápido.
- **Regra:** Sempre filtre as partições físicas (`year`, `month`, `date`) na PRIMEIRA CTE (`base_janelas`), antes de qualquer `JOIN`.
- **Anti-pattern:** Fazer um `LEFT JOIN` da tabela de TPV inteira com a tabela de contratos para só depois filtrar o mês atual na cláusula `WHERE` final.

## 2. Window Functions e Skewness
Cálculos de rateio (ex: CNPJ Master) podem causar Data Skew se um Grupo Econômico for infinitamente maior que os outros.
- **Regra:** Ao usar `ROW_NUMBER() OVER (PARTITION BY company_group_id)`, garanta que a granularidade esteja correta para evitar que uma única partição trave um nó do cluster.
- **Deduplicação:** Use `QUALIFY ROW_NUMBER() OVER (...) = 1` nativo do Databricks em vez de subqueries pesadas.

## 3. Broadcast Joins
Tabelas de configuração de impostos ou regras de negócio são pequenas.
- **Regra:** Tabelas menores que 10MB (como `contract_economic_index_rates`) devem ser sinalizadas para Broadcast no plano de execução para acelerar o cruzamento com a tabela massiva de eventos.