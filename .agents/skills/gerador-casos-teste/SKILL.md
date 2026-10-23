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

## 1. Técnicas Obrigatórias de Desenho de Testes (ISTQB v4.0 — Abordagem Caixa-Preta)

Não gere apenas testes óbvios. Aplique ativamente as seguintes técnicas sobre cada Critério de Aceite, mantendo **fidelidade estrita ao escopo da User Story**:

1. **Particionamento de Equivalência (EP):** Identifique classes de entradas válidas e inválidas (campos vazios, formatos incorretos, caracteres especiais não permitidos como `"@#$%"`, normalização de espaços e letras maiúsculas/minúsculas) sem criar testes redundantes para a mesma partição.
2. **Análise de Valor Limite (BVA):** Sempre que houver limites numéricos, monetários, prazos de tempo, tamanhos de campo ou limites de tentativas definidos na US, crie casos de teste para os **valores de fronteira**:
   - Limite inferior inválido (`mínimo - 1`), no limite exato (`mínimo`) e acima do limite (`máximo + 1`).
3. **Tabelas de Decisão e Transição de Estados:**
   - Combine regras de negócio interdependentes (ex.: cupom válido + produto não elegível + frete).
   - Teste mudanças de ciclo de vida e revogação dinâmica (ex.: benefício ativo -> utilizado -> expirado; remoção de item do carrinho que invalida regra previamente atendida).
4. **Validações de Segurança Funcional e Prevenção a Uso Indevido:**
   - **Restrições de Uso e Identidade:** Valide bloqueios de reutilização por CPF/usuário, acúmulo indevido de benefícios ou limites de tentativas **apenas quando previstos na User Story**.
   - **Simultaneidade sob a Ótica do Usuário:** Teste tentativas simultâneas (ex.: mesmo usuário com duas abas abertas ou dois clientes disputando o último uso disponível de um benefício) descrevendo o resultado funcional esperado na tela, sem jargões internos de banco de dados.
   - **Proibição de Scripts e Tags HTML (`<script>` / SQLi):** **NUNCA** insira tags HTML/JavaScript (`<script>alert(1)</script>`) ou comandos SQL (`' OR 1=1 --`) nos passos BDD ou resultados esperados. Para validar entradas malformadas, utilize apenas caracteres especiais comuns (ex.: `"CUPOM@#$%"`).
5. **Regra de Ouro — Não-Alucinação Técnica (Caixa-Preta):**
   - **NUNCA** invente códigos de status HTTP (`HTTP 200`, `400`, `404`, `409`, `422`, `429`, `500`), cabeçalhos de rede (`Retry-After`, `X-Forwarded-For`), objetos/campos JSON (`subtotal`, `discount`, `max_uses`) ou detalhes de implementação de banco de dados (`lock pessimista`, `TTL`, `queries parametrizadas`) se a User Story não especificar contratos técnicos de API.
   - Escreva os Casos de Teste sob a perspectiva de **comportamento observável pelo usuário e regras de negócio**.

---

## 2. Cobertura Obrigatória de Cenários

Agrupe os Casos de Teste nas três categorias abaixo:

- **Caminho Feliz**: Fluxos principais de sucesso onde todas as pré-condições e regras de negócio são atendidas.
- **Caminho Alternativo**: Validações de campos obrigatórios, formatos inválidos, valores limite (BVA), regras de negócio secundárias e revogações automáticas.
- **Exceção e Segurança**: Dados inexistentes/expirados/esgotados, tentativas de reutilização indevida (ex.: mesmo CPF), tentativas de burlar regras de negócio em sessões simultâneas e entradas com caracteres inválidos.

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

##### Caso de Teste: [USXX-CT-XXX] - [Título sucinto da ação e resultado esperado]

- **Tipo:** Funcional | Validação | Segurança | Exceção
- **Prioridade:** Alta | Média | Baixa
- **Pré-condições:** [Estado inicial necessário, massas de dados ou configurações prévias]
- **Passos de Execução (BDD):**
  **Dado** [contexto inicial e estado do sistema com dados concretos]
  **Quando** [ação executada pelo usuário ou requisição disparada]
  **E** [condição ou ação complementar, se aplicável]
  **Então** [resultado esperado observável no sistema]
- **Resultado Esperado Detalhado:** [Comportamento observável na interface, reflexo exato nos valores/estados do pedido e mensagens exatas exibidas ao usuário (não invente status HTTP, cabeçalhos, payloads JSON ou campos de banco não descritos na US)]
```

---

## 5. Checklist de Auto-Validação (Antes de Entregar)

Antes de finalizar a resposta, verifique internamente:

- [ ] Todos os códigos de Cenário e CT possuem o prefixo `USXX-` (`[USXX-CN-XX]` e `[USXX-CT-XXX]`)?
- [ ] A numeração de `[USXX-CT-XXX]` é sequencial e sem duplicatas ao longo de todo o documento?
- [ ] O campo `Tipo` contém apenas **um** valor permitido (`Funcional`, `Validação`, `Segurança` ou `Exceção`)?
- [ ] O campo `Prioridade` contém apenas **um** valor permitido (`Alta`, `Média` ou `Baixa`)?
- [ ] Todos os Critérios de Aceite da User Story estão cobertos e testados com valores limite (BVA)?
- [ ] O documento está **100% livre** de detalhes técnicos inventados (sem códigos HTTP, sem cabeçalhos de rede, sem JSON e sem jargões de banco) e **sem tags `<script>` ou comandos SQL**?
