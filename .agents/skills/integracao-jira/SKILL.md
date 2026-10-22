---
name: integracao-jira
description: >-
  Use esta skill sempre que precisar transformar especificações de Casos de Teste em Markdown
  em uma hierarquia de issues no Jira (Épico -> Task -> Sub-task) no projeto QAA utilizando
  o servidor Atlassian Remote MCP, garantindo idempotência e rastreabilidade completa.
---

# Skill: Integração Hierárquica de Testes no Jira (Atlassian MCP)

Ao receber um artefato Markdown de especificação de testes (ex.: `artifacts/casos_teste_*.md`), decomponha sua estrutura na hierarquia de **3 níveis** dentro do projeto **`QAA`** no Jira usando as ferramentas do servidor `atlassian` MCP.

---

## 1. Mapeamento da Hierarquia em 3 Níveis

Toda especificação de testes segue a relação pai-filho estrita abaixo no projeto **`QAA`**:

| Nível | Elemento no Markdown | Tipo de Issue no Jira | Vínculo Pai (`parent`) | Padrão do Campo `summary` |
| :--- | :--- | :--- | :--- | :--- |
| **Nível 1** | `# Especificação de Testes: USXX - [Título]` | `Epic` (ou `Épico`) | *(Nenhum — Raiz)* | `[USXX] - [Título da Funcionalidade]` |
| **Nível 2** | `### Cenário: [USXX-CN-XX] - [Título]` | `Task` (ou `Tarefa`) | Chave do Épico (`QAA-XX`) | `[USXX-CN-XX] - [Título do Cenário]` |
| **Nível 3** | `##### Caso de Teste: [USXX-CT-XXX] - [Título]` | `Sub-task` (ou `Subtarefa`) | Chave da Task do Cenário (`QAA-YY`) | `[USXX-CT-XXX] - [Título do Caso de Teste]` |

---

## 2. Procedimento Passo a Passo de Execução

### Passo 1: Descoberta de Contexto e Metadados (Pré-voo)

1. **Obter `cloudId`:** Chame a ferramenta `getAccessibleAtlassianResources` para descobrir o `cloudId` e a URL base do site Atlassian Cloud conectado.
2. **Inspecionar Tipos de Issue do Projeto `QAA`:** Chame `getJiraProjectIssueTypesMetadata` passando o `cloudId` e `projectIdOrKey: "QAA"` (ou `projectKey: "QAA"`).
   - Identifique os nomes/IDs exatos configurados no idioma do projeto para:
     - **Nível 1:** `Epic` / `Épico`
     - **Nível 2:** `Task` / `Tarefa`
     - **Nível 3:** `Sub-task` / `Subtarefa` / `Sub-tarefa`

### Passo 2: Verificação de Idempotência (Prevenção de Duplicatas)

Antes de criar qualquer issue, consulte os itens já registrados para a mesma User Story no projeto `QAA` usando `searchJiraIssuesUsingJql`:

```text
project = "QAA" AND summary ~ "USXX"
```

- Construa um mapa em memória com os itens já existentes:
  - Se o Épico `[USXX]` já existir, reutilize sua chave (`key`, ex.: `QAA-10`) sem criar outro Épico.
  - Se uma Task `[USXX-CN-XX]` já existir, reutilize sua chave (`key`, ex.: `QAA-11`) sem recriá-la.
  - Se uma Sub-task `[USXX-CT-XXX]` já existir, pule sua criação e registre a chave existente no relatório final.

### Passo 3: Criação Top-Down Respeitando Dependências (`createJiraIssue`)

Execute a criação estritamente na ordem **Nível 1 → Nível 2 → Nível 3**, pois cada nível depende da chave (`key`) gerada no nível imediatamente superior:

#### 3.1. Nível 1 — Criar (ou Reutilizar) o Épico da User Story
- **Parâmetros de `createJiraIssue`:**
  - `cloudId`: `<CLOUD_ID>`
  - `projectKey`: `"QAA"`
  - `issueTypeName`: `"Epic"` (ou nome equivalente retornado no Passo 1)
  - `summary`: `[USXX] - [Título da Funcionalidade]`
  - `description`:
    ```markdown
    ## Especificação de Testes da User Story USXX

    - **Funcionalidade:** [Título da Funcionalidade]
    - **Arquivo de Referência (Requisito):** `[caminho/do/requisito.md]`
    - **Artefato de Testes:** `[artifacts/casos_teste_usxx.md]`
    - **Total de Cenários:** [N]
    - **Total de Casos de Teste:** [M]
    ```
- Armazene a chave retornada como `EPIC_KEY` (ex.: `QAA-10`).

#### 3.2. Nível 2 — Criar (ou Reutilizar) as Tasks de cada Cenário
Para cada bloco `### Cenário: [USXX-CN-XX] - [Título do Cenário]`:
- **Parâmetros de `createJiraIssue`:**
  - `cloudId`: `<CLOUD_ID>`
  - `projectKey`: `"QAA"`
  - `issueTypeName`: `"Task"` (ou `"Tarefa"`)
  - `parent`: `<EPIC_KEY>`
  - `summary`: `[USXX-CN-XX] - [Título do Cenário]`
  - `description`:
    ```markdown
    ## Cenário de Teste: [USXX-CN-XX]

    - **Categoria:** [Caminho Feliz | Caminho Alternativo | Exceção e Segurança]
    - **Objetivo:** [Objetivo do cenário extraído do Markdown]

    ### Casos de Teste Vinculados (Sub-tasks):
    - `[USXX-CT-XXX]` - [Título do CT]
    ```
- Armazene a chave retornada no mapa `SCENARIO_TASKS["USXX-CN-XX"] = <TASK_KEY>`.

#### 3.3. Nível 3 — Criar as Sub-tasks de cada Caso de Teste
Para cada bloco `##### Caso de Teste: [USXX-CT-XXX] - [Título do Caso de Teste]` pertencente ao cenário `USXX-CN-XX`:
- **Parâmetros de `createJiraIssue`:**
  - `cloudId`: `<CLOUD_ID>`
  - `projectKey`: `"QAA"`
  - `issueTypeName`: `"Sub-task"` (ou `"Subtarefa"`)
  - `parent`: `<SCENARIO_TASKS["USXX-CN-XX"]>`
  - `summary`: `[USXX-CT-XXX] - [Título do Caso de Teste]`
  - `description`:
    ```markdown
    ## Especificação do Caso de Teste: [USXX-CT-XXX]

    - **Cenário Pai:** [USXX-CN-XX] - [Título do Cenário]
    - **Tipo:** [Funcional | Validação | Segurança | Exceção]
    - **Prioridade:** [Alta | Média | Baixa]
    - **Pré-condições:** [Pré-condições extraídas do Markdown]

    ### Passos de Execução (BDD)
    **Dado** [contexto inicial]
    **Quando** [ação realizada]
    **E** [condição complementar]
    **Então** [resultado esperado]

    ### Resultado Esperado Detalhado
    [Resultado esperado detalhado extraído do Markdown]
    ```

---

## 3. Regra Estrita de Formatação de "Passos de Execução (BDD)"

Assim como na integração com o Notion, **nunca concatene os passos BDD em uma única linha**. Cada instrução Gherkin (`**Dado**`, `**Quando**`, `**E**`, `**Então**`) deve permanecer em sua própria linha com quebra de linha (`\n`) e palavra-chave em negrito.

---

## 4. Formato de Relatório de Saída

Ao concluir a sincronização, retorne ao Orquestrador um resumo estruturado em árvore contendo todas as chaves e URLs criadas/reutilizadas:

```markdown
### Sincronização Jira Concluída (Projeto `QAA`)

- 🟣 **Épico:** [QAA-10](https://<site>.atlassian.net/browse/QAA-10) — `[US01] - Recuperação de Senha por E-mail`
  - 🔵 **Task (Cenário):** [QAA-11](https://<site>.atlassian.net/browse/QAA-11) — `[US01-CN-01] - Solicitação e Redefinição de Senha com Sucesso`
    - ⚪ **Sub-task (CT):** [QAA-12](https://<site>.atlassian.net/browse/QAA-12) — `[US01-CT-001] - Navegar para a tela de recuperação...`
    - ⚪ **Sub-task (CT):** [QAA-13](https://<site>.atlassian.net/browse/QAA-13) — `[US01-CT-002] - Solicitar recuperação de senha...`
```
