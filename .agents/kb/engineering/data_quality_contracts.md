# Padrões de Qualidade e Contratos de Dados (Billing)

## 1. Contratos de Dados na Camada Bronze (Data Contracts)
A camada Bronze é a porta de entrada. Falhas aqui propagam erros financeiros.
- **Schema Enforcement:** O tipo de dado da origem deve ser validado. Campos declarados como valores monetários nunca podem ser ingeridos como `STRING` sem um casting imediato e seguro (ex: `TRY_CAST`).
- **Completeness (Completude):** Colunas vitais para elegibilidade (ex: `company_nano_id`, `signed_at`, `status`) devem ter testes de `not_null`. Se o dado faltar, a linha deve ser isolada ou o pipeline deve falhar (fail-fast).

## 2. Padrões de Testes dbt (Silver & Gold)
O faturamento exige testes de lógica de negócio (Singular Tests) além dos testes genéricos de schema.
- **Unicidade Absoluta:** O grão final do faturamento (Gold) deve sempre passar no teste `unique` para a chave de negócio (ex: `contract_id` + `billing_month`).
- **Integridade Referencial (Relationships):** Todos os IDs faturados na tabela final devem existir na tabela de dimensão de clientes/contratos.
- **Accepted Values:** Campos de status ou categoria (ex: `billing_model`) devem ser testados contra uma lista restrita de valores aceitos.

## 3. Testes Singulares de Precisão Financeira
Os modelos devem ser acompanhados de queries de teste (`tests/`) que validem:
- **Zero-Sum Tests:** `(valor_bruto - descontos) = net_final`. Qualquer diferença > 0.01 deve falhar o teste.
- **Non-Negative Invoicing:** O `net_final` de uma fatura nunca pode ser menor que zero, a menos que seja explicitamente classificado como 'Nota de Crédito'.
- **Taxa de Variação (Anomaly Detection):** Testar se a contagem de faturas geradas no mês atual não caiu ou subiu mais de 20% em relação ao mês anterior de forma inexplicável.

## Diretrizes para o @financial-auditor
Ao revisar um pipeline de faturamento, o auditor deve exigir que o arquivo `schema.yml` reflita esses contratos e verificar se foram criados os testes singulares (queries SQL na pasta `tests/`) para garantir o Zero-Sum Test.