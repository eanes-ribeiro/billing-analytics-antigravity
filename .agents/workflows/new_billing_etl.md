---
description: description: Inicia a construção de um novo motor de faturamento do zero, orquestrando PM, Engenharia e Auditoria, com atualização automática da KB.
---


# Workflow: /new-billing-etl (Versão Atualizada)

When the user types `/new-billing-etl <descricao_da_demanda>`:

## Phase 1: Discovery & State Initialization
1. **Agent**: `@billing-pm`
2. **Action**: Utilize a skill `context-manager` para iniciar o arquivo `docs/state.md`. Em seguida, use a skill `billing_discovery`.
3. **Task**: Entreviste o usuário para mapear a hierarquia de descontos (Waterfall) e a janela de faturamento. Gere a especificação técnica em `docs/specs/billing_rule_<nome_da_regra>.md`.
4. **Halt**: PAUSE obrigatoriamente. Peça a aprovação explícita do usuário sobre as regras matemáticas e o documento gerado antes de acionar a engenharia.

## Phase 1.5: KB Update (Após aprovação do usuário)
1. **Agent**: `@billing-pm` (ou um novo agente `@kb-manager` para maior especialização)
2. **Action**: Utilize a skill `kb_updater` (uma nova skill ou uma extensão da `context-manager`).
3. **Task**: Com base na especificação técnica aprovada em `docs/specs/billing_rule_<nome_da_regra>.md`, extraia as regras de faturamento, hierarquia de descontos e janelas temporais, e adicione-as de forma estruturada à Base de Conhecimento (`.agents/kb/`). Por exemplo, as regras de impostos podem ir para `kb/tax_rules.md`, hierarquia de descontos para `kb/discount_hierarchy.md`, e janelas de faturamento para `kb/billing_cycles.md`.

## Phase 2: Engineering Execution (Somente após aprovação e atualização da KB)
1. **Agent**: `@analytics-engineer`
2. **Action**: Utilize a skill `dbt_engineering` (e `sql_optimization` se julgar necessário). **Deve consultar a KB atualizada** para padrões de faturamento e regras.
3. **Task**: Construa as CTEs do SQL em `src/models/` seguindo estritamente a especificação aprovada e as diretrizes da KB. Garanta a conversão correta para `DECIMAL(18,4)`.

## Phase 3: Financial Verification
1. **Agent**: `@financial-auditor`
2. **Action**: Utilize a skill `financial_verifier`. **Deve consultar a KB atualizada** para regras de negócio e padrões de auditoria.
3. **Task**: Audite o código gerado no passo anterior. Crie o teste de reconciliação cruzada (Soma da Entrada vs Soma da Saída).
4. **Finalization**: Se o teste não tiver risco de perda centesimal, use o `context-manager` para atualizar o status do projeto para "Finalizado" e registrar a regra na KB como "Validada".
