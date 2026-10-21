# QA Agents Workflow - Orquestrador Geral

Este projeto automatiza o fluxo de Garantia da Qualidade (QA) para:

1. Analisar requisitos ou User Stories.
2. Decompor as demandas em Cenários e Casos de Teste (CTs) estruturados em Gherkin.
3. Persistir as especificações em arquivos locais para versionamento e auditoria.
4. Documentar os Casos de Teste em base de dados no Notion via MCP.
5. Criar tarefas e tickets de execução no Jira correspondentes a cada CT *(quando aplicável/disponível)*.

## Equipe de Subagents

1. **qa-analyst** (`.agents/agents/qa-analyst.md`):
   - Especialista em Engenharia de QA.
   - Analisa especificações e gera cenários e casos de teste seguindo a skill `gerador-casos-teste`.
2. **notion-documenter** (`.agents/agents/notion-documenter.md`):
   - Faz o parse dos cenários/CTs locais e insere os casos de teste na tabela do Notion via MCP.
3. **jira-manager** *(Planejado / Sob Demanda)*:
   - Cria as issues/tarefas correspondentes a cada caso de teste no Jira (acionado apenas quando configurado em `.agents/agents/` e solicitado pelo usuário).

# Fluxo Padrão de Execução

Quando um objetivo envolver a geração de testes para uma User Story ou requisito:

1. **Etapa 1 - Ingestão:**
   - O agente orquestrador lê o arquivo de entrada com o requisito/User Story.
2. **Etapa 2 - Elaboração de testes:**
   - Aciona o subagente `qa-analyst` para estruturar os cenários e casos de teste cobrindo Caminho Feliz, Alternativo e Exceção/Segurança.
   - O `qa-analyst` retorna o conteúdo Markdown estruturado e o **Orquestrador** salva a especificação resultante no caminho `artifacts/casos_teste_<nome_da_us>.md`.
3. **Etapa 3 - Portão de Validação Humana (Human-in-the-Loop):**
   - Apresente um resumo quantitativo e os principais cenários gerados no terminal.
   - **PAUSE e solicite a confirmação explícita do usuário** utilizando a ferramenta `ask_question` (garantindo o bloqueio seguro mesmo em execuções com `/goal`) antes de disparar qualquer envio para ferramentas externas.
4. **Etapa 4 - Sincronização Externa (notion-documenter & jira-manager):**
   - Após a aprovação explícita do usuário:
     - Acione `notion-documenter` para popular a base de dados do Notion.
     - Acione `jira-manager` para abrir os tickets correspondentes no Jira *(somente se solicitado pelo usuário e se o subagente estiver ativo)*.
   - Retorne os links finais das páginas e tickets criados.

## Comandos e Gatilhos Recomendados

- **Execução do ciclo completo:**
  `/goal "Gere os casos de teste para o requisito em examples/user_story_login.md e sincronize com Notion e Jira."`
- **Apenas elaboração local (sem envio externo):**
  `/goal "Analise examples/user_story_login.md com o qa-analyst e salve em artifacts."`
- **Apenas sincronização de arquivo existente:**
  `/goal "Use o notion-documenter para sincronizar o arquivo artifacts/casos_teste_login.md com a tabela do Notion."`

---

## O que NÃO fazer (Guardrails e Restrições Críticas)

- **NUNCA envie dados para Notion ou Jira sem validação prévia:** O orquestrador jamais deve pular a Etapa 3.
- **NUNCA altere ou exclua testes já existentes:** Não apague tabelas ou limpe bases de dados sem autorização expressa.
- **NUNCA faça inserções parciais sem salvar o artefato:** O arquivo Markdown em `artifacts/` é a fonte da verdade e deve ser salvo antes de qualquer chamada de API.
- **NUNCA busque páginas aleatórias no Notion:** As operações no Notion devem se restringir estritamente à base de dados declarada no subagente.
