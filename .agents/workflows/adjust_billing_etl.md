---
description: description: Orquestra a refatoração ou alteração de regras de negócio em um ETL de faturamento existente, garantindo a integridade do legado e evitando regressão financeira.
---

# Workflow: /adjust-billing-etl

When the user types `/adjust-billing-etl <nome_do_etl> <descricao_da_mudanca>`:

## Phase 1: Discovery & Lineage
**Agent:** @code-explorer
**Skill:** explore_billing_logic
**Context:** - Diretório `models/` para análise de linhagem.
- Arquivo `README.md` do projeto para contexto de negócio.
**Goal:** Identificar todos os modelos impactados pela alteração solicitada e mapear a origem do dado até a camada Gold.

## Phase 2: Delta Specification
1. **Agent**: `@billing-pm`
2. **Action**: Utilize a skill `billing_discovery`.
3. **Task**: Com base no código mapeado e na `<descricao_da_mudanca>`, defina exatamente em qual etapa da cascata Waterfall a modificação ocorrerá. Atualize a especificação correspondente em `docs/specs/`.
4. **Halt**: PAUSE OBRIGATORIAMENTE. Valide o exemplo numérico do "Antes e Depois" da alteração com o usuário.

## Phase 3: Engineering Execution
1. **Agent**: `@analytics-engineer`
2. **Action**: Utilize a skill `dbt_engineering`. Acione o `@sql-optimizer` (skill `sql_optimization`) se o código legado possuir gargalos de performance identificados na Fase 1.
3. **Task**: Refatore os modelos em `src/models/`. Mantenha a compatibilidade com a arquitetura existente. Garanta a manutenção dos tipos `DECIMAL(18,4)` e a lógica de idempotência.

## Phase 4: Regression & Financial Audit
1. **Agent**: `@financial-auditor`
2. **Action**: Utilize a skill `financial_verifier`.
3. **Task**: Conduza Testes de Regressão Financeira. Valide se a alteração não quebrou a reconciliação (Soma Bronze vs Soma Gold) para o período atual e garanta que o histórico não foi afetado indevidamente.
4. **Finalization**: Se o teste não apresentar divergências centesimais, invoque o `context-manager` para atualizar o `docs/state.md` como "Concluído".