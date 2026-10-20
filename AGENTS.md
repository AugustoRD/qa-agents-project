# QA Agents Workflow - Orquestador Geral

Este projeto automatiza o fluxo de Garantia da Qualidade (QA): recebe requisitos ou User Stories, gera Casos de Teste (CTs) padronizados e, futuramente, documenta no Notion e abre tickets no Jira.

## Equipe de Subagents

1. **qa-analyst** (`.agents/agents/qa-analyst.md`):
   - Especialista em Engenharia de QA
   - Analisa especificações e gera cenários e casos de teste usando a skill `gerador-casos-teste`.
2. **notion-documenter** :
   - Formata e insere os casos de teste na tabela notion.
3. **jira-manager** :
   - Cria as issues/tarefas correspondentes a cada caso de teste no Jira.

# Fluxo Padrão de Execução

Quando um objetivo envolver a geração de testes para uma User Story ou requisito:

1. O agente orquestrador lê o arquivo de entrada com o requisito/User Story.
2. Aciona o subagente `qa-analyst` para estruturar os cenários e casos de teste cobrindo Caminho Feliz, Alternativo e Exceção/Segurança.
3. Garante que os casos de teste sigam rigorosamente a estrutura da skill `gerador-casos-teste`.
4. Exibe os Casos de Testes gerados para validação do usuário antes de enviar para ferramentas externas.
