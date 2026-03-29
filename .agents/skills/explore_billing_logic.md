# Skill: Explore Billing Logic

## Description
Especialista em rastreamento de linhagem de dados (data lineage) e descoberta de regras de negócio em modelos dbt de faturamento. Esta skill permite navegar entre as camadas Bronze, Silver e Gold para identificar transformações críticas.

## Instructions
1. **Mapeamento de Linhagem**: Ao analisar uma tabela Gold, identifique todas as fontes (sources) e modelos intermediários (refs) consultando os arquivos `.sql` e arquivos de configuração `.yml`.
2. **Rastreamento de Tipagem**: Verifique em qual camada ocorre a conversão dos dados financeiros para `DECIMAL(18,4)`. Se encontrar tipos `FLOAT` ou `DOUBLE` em camadas de cálculo, sinalize como erro grave.
3. **Identificação de Regras de Pro-rata**: Busque por lógicas que envolvam janelas temporais e cruzamento com snapshots dbt (SCD Tipo 2) para entender como alterações contratuais retroativas são tratadas.
4. **Descoberta de Dependências**: Antes de sugerir mudanças, liste todos os modelos "downstream" que serão impactados pela alteração na lógica de faturamento.

## Tools
- `file_search`: Para localizar referências de tabelas em todo o diretório `models/`.
- `code_interpreter`: Para simular pequenas transformações lógicas e validar cálculos de rateio.

## Stop Conditions
- A tarefa é considerada concluída quando o agente apresenta um diagrama ou lista clara da linhagem, do ponto de origem até o campo final na fatura.