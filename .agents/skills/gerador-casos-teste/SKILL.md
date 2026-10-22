---
name: gerador-casos-teste
description: >-
  Use esta skill sempre que precisar analisar User Stories, critérios de aceite ou requisitos
  funcionais/não-funcionais para desenhar Cenários de Teste e Casos de Teste detalhados em
  Gherkin (BDD), garantindo cobertura técnica (ISTQB/OWASP).
---

# Skill: Geração Padronizada de Casos de Teste (QA Engineering)

Ao analisar uma especificação de requisito ou User Story (ex.: `US01`, `US02`), decomponha a demanda em **Cenários de Teste (alto nível)** e **Casos de Teste detalhados (baixo nível)** aplicando engenharia de testes sistemática.

---

## 1. Técnicas Obrigatórias de Desenho de Testes (ISTQB v4.0 & OWASP)

Não gere apenas testes óbvios. Aplique ativamente as seguintes técnicas sobre cada Critério de Aceite:

1. **Particionamento de Equivalência (EP):** Identifique classes de entradas válidas e inválidas (campos vazios, formatos incorretos, tipos incompatíveis, normalização como `trim`/`lowercase`) sem criar testes redundantes para a mesma partição.
2. **Análise de Valor Limite (BVA):** Sempre que houver limites numéricos, monetários, prazos de tempo, tamanhos de campo ou limites de tentativas, crie casos de teste para os **valores de fronteira**:
   - Limite inferior inválido (`mínimo - 1`), no limite exato (`mínimo`) e acima do limite (`máximo + 1`).
3. **Tabelas de Decisão e Transição de Estados:**
   - Combine regras de negócio interdependentes (ex.: cupom válido + produto não elegível + frete).
   - Teste mudanças de ciclo de vida e revogação dinâmica (ex.: token ativo -> utilizado -> expirado; remoção de item do carrinho que invalida regra previamente atendida).
4. **Heurísticas de Segurança e Confiabilidade (OWASP):**
   - **Prevenção a Abuso e Força Bruta:** Rate limiting por IP/conta/sessão (`HTTP 429`).
   - **Privacidade e Enumeração:** Mensagens e tempos de resposta idênticos para evitar enumeração de usuários/recursos.
   - **Concorrência (Race Conditions):** Requisições simultâneas disputando recursos de uso único ou saldo/limite global.
   - **Injeção e Sanitização:** Tratamento seguro de payloads maliciosos (SQLi, XSS) sem expor stack traces ou erros internos (`HTTP 500`).

---

## 2. Cobertura Obrigatória de Cenários

Agrupe os Casos de Teste nas três categorias abaixo:

- **Caminho Feliz**: Fluxos principais de sucesso onde todas as pré-condições e regras de negócio são atendidas.
- **Caminho Alternativo**: Validações de campos obrigatórios, formatos inválidos, valores limite (BVA), regras de negócio secundárias e revogações automáticas.
- **Exceção e Segurança**: Dados inexistentes/expirados/esgotados, tentativas de reutilização indevida, bloqueios de segurança, concorrência e ataques de injeção/força bruta.

---

## 3. Regras Estritas de Contrato (Compatibilidade com Notion e Jira)

Para que os subagentes `notion-documenter` e `jira-manager` consigam sincronizar os testes sem erros de schema ou quebra da hierarquia de 3 níveis no Jira (**Épico → Task → Sub-task**), **obedeça rigorosamente**:

1. **Hierarquia Estrutural e Prefixo da User Story Obrigatório:** Todos os cabeçalhos e códigos devem preservar os 3 níveis hierárquicos e incluir o identificador da User Story em caixa alta:
   - **Nível 1 (User Story / Épico no Jira):** `# Especificação de Testes: USXX - [Título da Funcionalidade]`
   - **Nível 2 (Cenário / Task no Jira):** `### Cenário: [USXX-CN-XX] - [Título do Cenário]` (ex.: `[US01-CN-01]`, `[US02-CN-03]`).
   - **Nível 3 (Caso de Teste / Sub-task no Jira e Página no Notion):** `##### Caso de Teste: [USXX-CT-XXX] - [Título do Caso de Teste]` com 3 dígitos sequenciais (ex.: `[US01-CT-001]`, `[US02-CT-012]`).
   - _(Caso o requisito não informe o número da US, adote `US01` por padrão)._
2. **Valores Fechados de `Tipo` (Select Único):** Use **exatamente um** dos 4 valores abaixo por CT (proibido combinar ou criar novos):
   - `Funcional` | `Validação` | `Segurança` | `Exceção`
3. **Valores Fechados de `Prioridade` (Select Único baseado em Risco):** Use **exatamente um** dos 3 valores abaixo:
   - `Alta` (fluxo principal crítico, perda financeira, segurança ou bloqueio)
   - `Média` (validações de contorno, regras secundárias)
   - `Baixa` (formatação cosmética, normalização de espaços/caixa)
4. **Regras de Escrita Gherkin (BDD):**
   - **1 Comportamento por CT:** Não encadeie múltiplos pares `Quando... Então... Quando... Então` no mesmo CT.
   - **Dados Concretos:** Use valores explícitos nos passos (ex.: `"R$ 99,99"`, `"15 minutos"`, `"usuario@dominio.com"`).
   - **Formatação Multi-linha:** Cada passo deve estar em sua própria linha com a palavra-chave em negrito (`**Dado**`, `**Quando**`, `**E**`, `**Então**`).

---

## 4. Template Oficial de Saída (Markdown)

Gere o documento indo direto aos Cenários e Casos de Teste, seguindo exatamente esta estrutura:

```markdown
# Especificação de Testes: USXX - [Título da Funcionalidade]

**Referência:** `[caminho/do/arquivo_de_requisito.md]`

---

### Cenário: [USXX-CN-XX] - [Título do Cenário em Alto Nível]

- **Categoria:** Caminho Feliz | Caminho Alternativo | Exceção e Segurança
- **Objetivo:** [Explicação concisa do que este cenário avalia]

**Caso de Teste (CT):**

##### Caso de Teste: [USXX-CT-XXX] - [Título sucinto da ação e resultado esperado]

- **Tipo:** Funcional | Validação | Segurança | Exceção
- **Prioridade:** Alta | Média | Baixa
- **Pré-condições:** [Estado inicial necessário, massas de dados ou configurações prévias]
- **Passos de Execução (BDD):**
  **Dado** [contexto inicial e estado do sistema com dados concretos]
  **Quando** [ação executada pelo usuário ou requisição disparada]
  **E** [condição ou ação complementar, se aplicável]
  **Então** [resultado esperado observável no sistema]
- **Resultado Esperado Detalhado:** [Comportamento exato da interface, status HTTP/payload da API, persistência no banco e mensagens exatas exibidas]
```

---

## 5. Checklist de Auto-Validação (Antes de Entregar)

Antes de finalizar a resposta, verifique internamente:

- [ ] Todos os códigos de Cenário e CT possuem o prefixo `USXX-` (`[USXX-CN-XX]` e `[USXX-CT-XXX]`)?
- [ ] A numeração de `[USXX-CT-XXX]` é sequencial e sem duplicatas ao longo de todo o documento?
- [ ] O campo `Tipo` contém apenas **um** valor permitido (`Funcional`, `Validação`, `Segurança` ou `Exceção`)?
- [ ] O campo `Prioridade` contém apenas **um** valor permitido (`Alta`, `Média` ou `Baixa`)?
- [ ] Todos os Critérios de Aceite da User Story estão cobertos e testados com valores limite (BVA)?
