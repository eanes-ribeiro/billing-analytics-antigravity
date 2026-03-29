---
description: description: Atalho direto para auditar uma query SQL focando em otimização de performance e integridade financeira, ignorando a fase de planejamento.
---

# Workflow: /audit-query

When the user types `/audit-query <arquivo.sql>`:

## Phase 1: Performance & Architecture Review
1. **Agent**: `@sql-optimizer`
2. **Action**: Leia o conteúdo do `<arquivo.sql>`. Invoque a skill `sql_optimization`.
3. **Task**: 
   - Revise o plano de execução teórico da query.
   - Valide se o `Static Partition Pruning` (filtro da janela de faturamento, ex: 26 a 25) está aplicado no nível mais baixo (early filtering).
   - Confirme se não há `DISTINCT` pesados que possam ser substituídos por `ROW_NUMBER()`.
4. **Output**: Gere um breve relatório de otimização (Query Plan Analysis) propondo melhorias de performance, se necessário.

## Phase 2: Financial Integrity Audit
1. **Agent**: `@financial-auditor`
2. **Action**: Assuma o contexto. Analise a query original e as sugestões da Fase 1 usando a skill `financial_verifier`.
3. **Task**: 
   - **Precision Check**: Garanta que há ABSOLUTAMENTE ZERO uso de `FLOAT` ou `DOUBLE` e que cálculos intermediários usam `DECIMAL(18,4)`.
   - **Leakage Check**: Busque por `INNER JOINs` perigosos que possam dropar linhas de clientes elegíveis silenciosamente.
   - **Rounding Check**: Verifique se arredondamentos estão sendo feitos precocemente, o que poderia acumular distorções no Net Final.
4. **Halt**: PAUSE OBRIGATORIAMENTE. Emita um "LAUDO DE AUDITORIA" (Aprovado / Risco Encontrado) e exiba na tela para o usuário.

## Phase 3: State Sync (Background)
1. **Agent**: `@financial-auditor`
2. **Action**: Se a auditoria gerar ações de correção que o usuário decida implementar, utilize a skill `context-manager` para registrar a revisão no arquivo `docs/state.md`.