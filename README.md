# QA Agents Project — Automação de Engenharia de Testes com Notion & Jira MCP

Arquitetura multi-agente de Garantia da Qualidade (QA) construída para o **Google Antigravity CLI**, capaz de analisar User Stories e Critérios de Aceite, elaborar Cenários e Casos de Teste padronizados em Gherkin (BDD) com cobertura técnica, e sincronizar automaticamente a documentação com **Notion** e **Jira** via **Model Context Protocol (MCP)**.

---

## Arquitetura e Componentes

### 1. Orquestrador Geral (`AGENTS.md`)

Coordena o fluxo em 4 etapas com validação humana obrigatória antes de realizar qualquer escrita em ferramentas externas:

1. **Ingestão:** Leitura do requisito / User Story (ex.: `examples/user_story_login.md`).
2. **Elaboração de Testes:** Delegação ao subagente `qa-analyst` e persistência do artefato local em `artifacts/casos_teste_<nome_da_us>.md`.
3. **Portão de Validação Humana:** Apresentação do resumo executivo e pausa para aprovação explícita do usuário.
4. **Sincronização Externa:** Acionamento de `notion-documenter` e/ou `jira-manager`.

### 2. Subagents (`.agents/agents/`)

| Subagent                | Arquivo                               | Skills Vinculadas     |        MCP        | Responsabilidade                                                                                                                                                                            |
| :---------------------- | :------------------------------------ | :-------------------- | :---------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`qa-analyst`**        | `.agents/agents/qa-analyst.md`        | `gerador-casos-teste` |        Não        | Analisa requisitos e desenha Cenários (`[USXX-CN-XX]`) e Casos de Teste (`[USXX-CT-XXX]`) em Gherkin cobrindo Caminho Feliz, Alternativo e Exceção/Segurança.                               |
| **`notion-documenter`** | `.agents/agents/notion-documenter.md` | —                     |  Sim (`notion`)   | Lê os artefatos Markdown em `artifacts/` e registra os Casos de Teste de forma idempotente e em lote na base de dados do Notion.                                                            |
| **`jira-manager`**      | `.agents/agents/jira-manager.md`      | `integracao-jira`     | Sim (`atlassian`) | Lê os artefatos Markdown em `artifacts/` e estrutura as issues no projeto **`QAA`** do Jira em 3 níveis hierárquicos (**Épico → Task → Sub-task**) com verificação de idempotência via JQL. |

### 3. Skills (`.agents/skills/`)

- **`gerador-casos-teste`** (`.agents/skills/gerador-casos-teste/SKILL.md`):
  Diretrizes de engenharia de testes (Particionamento de Equivalência, Análise de Valor Limite, Tabelas de Decisão, Transição de Estados e Heurísticas OWASP) e template oficial de saída compatível tanto com o Notion quanto com a hierarquia de 3 níveis do Jira.
- **`integracao-jira`** (`.agents/skills/integracao-jira/SKILL.md`):
  Procedimento passo a passo para descoberta de `cloudId` e metadados de Issue Types, prevenção de duplicatas via JQL e criação top-down de issues no projeto `QAA`.

---

## Hierarquia de Issues no Jira (Projeto `QAA`)

O `jira-manager` mapeia cada especificação de testes na seguinte árvore dentro do projeto **`QAA`**:

- **Nível 1 — Épico (`Epic` / `Épico`):** Representa a User Story (`[USXX] - [Título da Funcionalidade]`).
  - **Nível 2 — Task (`Task` / `Tarefa`):** Representa cada Cenário de Teste (`[USXX-CN-XX] - [Título do Cenário]`), vinculado como filho (`parent`) do Épico.
    - **Nível 3 — Sub-task (`Sub-task` / `Subtarefa`):** Representa cada Caso de Teste (`[USXX-CT-XXX] - [Título do Caso de Teste]`), vinculado como filho (`parent`) da Task do seu respectivo Cenário.

---

## Configuração dos Servidores MCP (`~/.gemini/config/mcp_config.json`)

Certifique-se de que os servidores `notion` e `atlassian` estão configurados no seu `~/.gemini/config/mcp_config.json`:

```json
{
  "mcpServers": {
    "notion": {
      "serverUrl": "https://mcp.notion.com/mcp",
      "tools": {
        "notion-create-pages": {
          "eager": true
        },
        "notion-fetch": {
          "eager": true
        }
      }
    },
    "atlassian": {
      "serverUrl": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

---

## Exemplos de Uso

- **Ciclo Completo (Geração + Notion + Jira):**
  ```text
  /goal "Gere os casos de teste para o requisito em examples/user_story_login.md e sincronize com Notion e Jira."
  ```
- **Apenas Elaboração Local:**
  ```text
  /goal "Analise examples/us02_cupom_desconto.md com o qa-analyst e salve em artifacts."
  ```
- **Sincronizar Artefato Existente com o Jira (Projeto `QAA`):**
  ```text
  /goal "Use o jira-manager para sincronizar o arquivo artifacts/casos_teste_login.md com o projeto QAA no Jira."
  ```
- **Sincronizar Artefato Existente com o Notion:**
  ```text
  /goal "Use o notion-documenter para sincronizar o arquivo artifacts/casos_teste_login.md com a tabela do Notion."
  ```
