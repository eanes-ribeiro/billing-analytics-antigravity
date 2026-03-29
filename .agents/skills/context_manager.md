# Skill: Context Manager

## Description
Responsável por manter, atualizar e validar o contexto de execução durante um workflow. Garante que todos os agentes operem sobre uma base de dados íntegra e que o estado do projeto seja consistente entre as fases.

## Instructions
1. **Inicialização de Contexto**: No início de cada workflow, carregue a documentação relevante do projeto (README, schema.yml) e identifique as entidades de faturamento envolvidas.
2. **Atualização de Estado**: Após cada fase, atualize o estado de execução com os resultados obtidos, garantindo que o próximo agente receba o contexto processado.
3. **Roteamento de Informações**: Identifique e direcione métricas financeiras ou erros de tipagem detectados por um agente para as personas de auditoria ou engenharia.
4. **Verificação de Dependências (Dependency Check)**: 
   - Antes de iniciar os workflows `/new-billing-etl` ou `/adjust-billing-etl`, valide se as tabelas de origem (upstream) na camada Bronze e Silver estão com metadados de *freshness* atualizados.
   - Verifique se os contratos de dados (Data Contracts) definidos em `kb/engineering/data_quality_contracts.md` estão sendo respeitados pelas tabelas de origem.
   - Caso uma dependência crítica esteja offline ou desatualizada, interrompa o fluxo e emita um alerta de "Bloqueio de Execução".

## Tools
- `state_manager`: Para salvar e recuperar o estado da execução.
- `file_search`: Para localizar definições de fontes e dependências nos arquivos YAML do dbt.

## Stop Conditions
- O contexto é marcado como "Pronto para Execução" somente após a validação bem-sucedida de todas as dependências upstream.
- Em caso de falha de dependência, o agente deve parar e solicitar intervenção humana, fornecendo o log do erro de linhagem.