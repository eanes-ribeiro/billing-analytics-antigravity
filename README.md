# Billing Analytics Team - Antigravity

Este repositório define a estrutura e as operações de uma equipe de **Agentes de IA especializados em Billing Analytics** para a Antigravity. Meu objetivo é automatizar e otimizar processos críticos de faturamento, desde a definição de regras de negócio até a engenharia de dados e auditoria financeira, garantindo precisão, integridade e conformidade.

## 1. Conceitos Fundamentais

Para entender a operação deste ecossistema, é essencial compreender os seguintes conceitos:

*   **Agentes**: Entidades autônomas com objetivos, habilidades (skills) e restrições bem definidas. Eles colaboram entre si para executar tarefas complexas.
*   **Skills**: Conjuntos de capacidades ou ferramentas que um agente pode utilizar. São as "ferramentas" que os agentes têm à disposição para cumprir seus objetivos.
*   **Workflows**: Sequências orquestradas de tarefas que definem como os agentes interagem para alcançar um objetivo maior. Eles guiam a colaboração e a tomada de decisão.
*   **Base de Conhecimento (KB)**: Um repositório centralizado de informações, regras de negócio, padrões técnicos e diretrizes. É o "cérebro" coletivo dos agentes, alimentado e consultado dinamicamente para garantir consistência e aprendizado contínuo.

## 2. Arquitetura da Equipe de Agentes

A equipe é composta por agentes especialistas, cada um com um papel crucial no ciclo de vida do faturamento. A colaboração é a chave, com a Base de Conhecimento atuando como um elo central para a disseminação de informações e regras.

| Agente | Papel Principal | Objetivo (Goal) | Skills Chave | Restrições Notáveis |
| :--- | :--- | :--- | :--- | :--- |
| **@billing-pm** | Arquiteto de Regras & Gerente de KB | Traduzir regras fiscais e de faturamento em especificações técnicas infalíveis e garantir a atualização da KB. | `billing_discovery`, `context_manager`, `kb_updater` | Deve conduzir discovery, definir hierarquia de descontos, validar regras com exemplos numéricos e **atualizar a KB após aprovação**. |
| **@analytics-engineer** | Construtor de Modelos | Desenvolver modelos dbt/SQL modulares seguindo a arquitetura Medallion e lógica temporal de faturamento. | `dbt_engineering`, `sql_optimization` | Uso obrigatório de CTEs, `DECIMAL(18,4)` para monetário, idempotência, documentação YAML e **consulta obrigatória à KB**. |
| **@financial-auditor** | Guardião da Receita | Garantir a integridade total da cascata de faturamento (Waterfall) e evitar perda de receita (revenue leakage). | `financial_verifier` | Reconciliação financeira, validação da integridade da cascata, verificação de duplicados/nulos e **consulta obrigatória à KB**. |
| **@code-explorer** | Investigador de Legado | Mapear linhagem de dados e identificar impactos em ETLs de faturamento legados. | `explore_billing_logic`, `billing_discovery` | Atua apenas em modo leitura, deve apresentar mapa de dependências antes de sugerir alterações. |
| **@sql-optimizer** | Especialista em Performance | Otimizar planos de execução e reduzir custos computacionais em motores de faturamento. | `sql_optimization` | Threshold de confiança de 0.90, escalar para `@analytics-engineer` se a otimização exigir qualquer alteração em regras de negócio. |

## 3. Skills Detalhadas

As skills são as ferramentas que capacitam nossos agentes. Abaixo, destacamos algumas das mais importantes:

*   **`billing_discovery`**: Habilidade para interagir com usuários e extrair requisitos de faturamento, mapeando regras de negócio complexas.
*   **`kb_updater`**: **(NOVA SKILL)** Essencial para a gestão dinâmica da KB. Permite que um agente extraia informações de especificações técnicas aprovadas e as insira de forma estruturada nos arquivos da Base de Conhecimento, garantindo rastreabilidade e consistência.
*   **`dbt_engineering`**: Capacidade de construir e manter modelos de dados usando dbt (data build tool), seguindo padrões de engenharia de dados e arquitetura Medallion.
*   **`financial_verifier`**: Habilidade para realizar auditorias financeiras, reconciliação de dados cross-layer e investigação de falhas de cálculo, garantindo a integridade da cascata de faturamento.
*   **`explore_billing_logic`**: Permite que o agente navegue e compreenda a lógica de sistemas de faturamento legados, identificando dependências e impactos.
*   **`sql_optimization`**: Foco na otimização de queries SQL para melhorar a performance e reduzir custos computacionais.

## 4. Workflows de Operação

Os workflows definem a coreografia entre os agentes, garantindo que as tarefas sejam executadas de forma sequencial e colaborativa. O "Human-in-the-loop" é mantido em pontos críticos para aprovação e validação.

### Workflow: `/new-billing-etl` (Criação de Novo ETL de Faturamento)

Este workflow orquestra a criação de um novo motor de faturamento, desde a concepção da regra até a auditoria final, com a atualização automática da KB:

1.  **Discovery & Especificação**: O `@billing-pm` entrevista o usuário, mapeia a hierarquia de descontos e a janela de faturamento, gerando a especificação técnica em `docs/specs/billing_rule_<nome_da_regra>.md`. Uma **PAUSA obrigatória** ocorre aqui, aguardando a aprovação explícita do usuário sobre as regras matemáticas e o documento gerado.
2.  **Atualização da KB**: **Após a aprovação do usuário**, o `@billing-pm` utiliza a skill `kb_updater` para extrair as regras da especificação e inseri-las nos arquivos relevantes da Base de Conhecimento (ex: `kb/tax_rules.md`, `kb/discount_hierarchy.md`, `kb/billing_cycles.md`).
3.  **Desenvolvimento de Engenharia**: O `@analytics-engineer` constrói os modelos dbt/SQL em `src/models/`, **consultando a KB atualizada** para padrões e regras, garantindo a conversão correta para `DECIMAL(18,4)`.
4.  **Verificação Financeira**: O `@financial-auditor` audita o código gerado, **consultando a KB atualizada** para regras de negócio e padrões de auditoria, criando testes de reconciliação cruzada. O processo só avança se a divergência for zero ou justificada por arredondamentos mínimos.
5.  **Finalização**: O status do projeto é atualizado para "Finalizado" ou "Em Produção" após a validação.

### Workflow: `/audit-leakage` (Auditoria de Perda de Receita)

Este workflow é acionado para investigar e corrigir anomalias que possam levar à perda de receita:

1.  **Varredura e Detecção**: O `@financial-auditor` proativamente busca por anomalias (valores nulos, negativos, inconsistências) ou divergências na reconciliação diária/mensal.
2.  **Investigação de Causa Raiz**: O `@code-explorer` é acionado para rastrear a linhagem de dados de qualquer valor problemático, identificando a origem do erro em ETLs ou sistemas legados.
3.  **Proposta de Correção**: Com base na investigação, o `@analytics-engineer` propõe e implementa o ajuste necessário no ETL ou na lógica de faturamento.
4.  **Validação da Correção**: O `@financial-auditor` re-executa os testes para garantir que a correção resolveu o problema sem introduzir novas inconsistências.

## 5. Base de Conhecimento (KB) - O Cérebro Dinâmico

A Base de Conhecimento (`.agents/kb/`) é o coração do aprendizado e da consistência dos agentes. Ela é alimentada dinamicamente e consultada por todos os agentes para garantir que operem com as informações mais atualizadas e padronizadas. Documentos essenciais incluem:

*   `tax_rules.md`: Catálogo abrangente de alíquotas e regras tributárias por região, tipo de serviço e período de vigência.
*   `discount_hierarchy.md`: Definição clara da precedência de aplicação de descontos.
*   `billing_cycles.md`: Detalhes sobre calendários de fechamento, janelas de competência e regras de pro-rata.
*   `data_quality_contracts.md`: Contratos de qualidade de dados para as camadas Bronze, Silver e Gold.
*   `dbt_billing_patterns.md`: Padrões de modelagem dbt específicos para faturamento.
*   `spark_sql_performance.md`: Diretrizes e dicas de otimização para queries Spark SQL.

Com a skill `kb_updater` e a diretriz para o `@billing-pm` de utilizá-la, a alimentação desses documentos se torna parte integrante do workflow de faturamento, garantindo que a KB reflita sempre o estado mais atual das regras de negócio.

## 6. Como Utilizar e Contribuir

Este repositório é a espinha dorsal para a automação inteligente do faturamento. Para utilizar ou contribuir:

1.  **Clonar o Repositório**: `git clone eanes-ribeiro/billing-analytics-antigravity`
2.  **Entender os Agentes**: Revise os arquivos em `.agents/agents.md` para compreender os papéis e responsabilidades.
3.  **Explorar Skills**: Consulte `.agents/skills/` para ver as capacidades de cada agente.
4.  **Executar Workflows**: Siga as instruções nos arquivos `.agents/workflows/` para iniciar e gerenciar tarefas de faturamento.
5.  **Contribuir para a KB**: Garanta que novas regras de negócio e padrões técnicos sejam documentados na KB, preferencialmente através do workflow `/new-billing-etl` que automatiza essa atualização.

## Referências

[1] Data Engineer Academy. (2025). *System Design for Data Engineers: AI Agents Architecture*. Disponível em: [https://dataengineeracademy.com/blog/system-design-for-data-engineers-ai-agents-architecture/](https://dataengineeracademy.com/blog/system-design-for-data-engineers-ai-agents-architecture/)
[2] Oracle Blogs. (s.d.). *How to Write Effective Instructions for AI Agents in Oracle Analytics Cloud*. Disponível em: [https://blogs.oracle.com/analytics/how-to-write-effective-instructions-for-ai-agents-in-oracle-analytics-cloud](https://blogs.oracle.com/analytics/how-to-write-effective-instructions-for-ai-agents-in-oracle-analytics-cloud)
[3] GetMonetizely. (2025). *How to Build Custom Billing Systems for AI Agents*. Disponível em: [https://www.getmonetizely.com/articles/how-to-build-custom-billing-systems-for-ai-agents-a-complete-guide](https://www.getmonetizely.com/articles/how-to-build-custom-billing-systems-for-ai-agents-a-complete-guide/)
[4] The Noah AI. (2026). *Streamline Financial Audits with AI Agents*. Disponível em: [https://www.thenoah.ai/resources/blogs/automating-financial-audits-with-agent-orchestration-and-zerocode-workflows](https://www.thenoah.ai/resources/blogs/automating-financial-audits-with-agent-orchestration-and-zerocode-workflows/)
[5] Fieldguide. (2026). *How to Use AI Agents for Financial Reporting Audits*. Disponível em: [https://www.fieldguide.io/resource-articles/ai-agents-for-financial-reporting-audits-guide](https://www.fieldguide.io/resource-articles/ai-agents-for-financial-reporting-audits-guide/)
