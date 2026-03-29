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

## Phase 2: Optimization Analysis
**Agent:** @sql-optimizer
**Skill:** sql_optimization
**Context:**
- Arquivo `kb/engineering/spark_sql_performance.md` para diretrizes de custo.
**Goal:** Identificar gargalos de performance, sugerir Broadcast Joins e garantir o Partition Pruning.

## Phase 3: State Sync (Background)
1. **Agent**: `@financial-auditor`
2. **Action**: Se a auditoria gerar ações de correção que o usuário decida implementar, utilize a skill `context-manager` para registrar a revisão no arquivo `docs/state.md`.