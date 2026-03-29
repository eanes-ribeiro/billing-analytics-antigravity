---
description: description: Inicia a construção de um novo motor de faturamento do zero, orquestrando PM, Engenharia e Auditoria.
---

# Workflow: /new-billing-etl

When the user types `/new-billing-etl <descricao_da_demanda>`:

## Phase 1: Discovery & State Initialization
1. **Agent**: `@billing-pm`
2. **Action**: Utilize a skill `context-manager` para iniciar o arquivo `docs/state.md`. Em seguida, use a skill `billing_discovery`.
3. **Task**: Entreviste o usuário para mapear a hierarquia de descontos (Waterfall) e a janela de faturamento. Gere a especificação técnica em `docs/specs/`.
4. **Halt**: PAUSE obrigatoriamente. Peça a aprovação explícita do usuário sobre as regras matemáticas e o documento gerado antes de acionar a engenharia.

## Phase 2: Engineering Execution (Somente após aprovação)
1. **Agent**: `@analytics-engineer`
2. **Action**: Utilize a skill `dbt_engineering` (e `sql_optimization` se julgar necessário).
3. **Task**: Construa as CTEs do SQL em `src/models/` seguindo estritamente a especificação aprovada. Garanta a conversão correta para `DECIMAL(18,4)`.

## Phase 3: Financial Verification
1. **Agent**: `@financial-auditor`
2. **Action**: Utilize a skill `financial_verifier`.
3. **Task**: Audite o código gerado no passo anterior. Crie o teste de reconciliação cruzada (Soma da Entrada vs Soma da Saída).
4. **Finalization**: Se o teste não tiver risco de perda centesimal, use o `context-manager` para atualizar o status do projeto para "Finalizado".