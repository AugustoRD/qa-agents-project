---
name: gerador-casos-teste
description: Diretrizes e padrões para a geração de Casos de Teste (CTs) a partir de User Stories ou requisitos.
---

# Skill: Geração Padronizada de Casos de Teste

Ao analisar qualquer especificação de requisito ou User Story (ex: `US01`, `US02`), você deve primeiro identificar os **Cenários de Teste (alto nível)** e, dentro de cada cenário, derivar os **Casos de Teste detalhados (baixo nível)**, prefixando sempre os códigos com o identificador da User Story (`[USXX-CN-XX]` e `[USXX-CT-XXX]`).

### Cobertura Obrigatória de Cenários:

- **Caminho Feliz (Happy Path)**: Fluxos de sucesso, onde o usuário segue o processo sem erros ou exceções.
- **Caminhos Alternativos (Alternative Paths)**: Fluxos de validação de campos obrigatórios, formatos inválidos, regras de negócio e outras variações que não interrompam o processo principal.
- **Caminhos de Exceção e Segurança (Exception/Security Paths)**: Fluxos que lidam com erros, falhas de autenticação, tentativas de acesso não autorizado, limites de tentativas e outras situações que possam comprometer a segurança ou integridade do sistema.

## Formato Padrão de Casos de Teste (CTs)

Estruture a resposta agrupando os Casos de Teste sob os seus respectivos Cenários:

### Cenário: [USXX-CN-XX] - [Título do Cenário em Alto Nível]

- **Categoria:** Caminho Feliz | Caminho Alternativo | Exceção e Segurança
- **Objetivo:** [Explicação concisa do que este cenário avalia]

**Caso de Teste (CT):**

##### Caso de Teste: [USXX-CT-XXX] - [Título sucinto da ação e resultado esperado]

- **Tipo:** Funcional | Validação | Segurança | Exceção
- **Prioridade:** Alta | Média | Baixa
- **Pré-condições:** [O que deve estar preparado ou configurado antes da execução]
- **Passos de Execução (BDD):**
  - **Dado** [contexto inicial e estado do sistema]
  - **Quando** [ação executada pelo usuário com dados específicos]
  - **E** [ações ou condições adicionais, se aplicável]
  - **Então** [resultado esperado observável no sistema]
- **Resultado Esperado Detalhado:** [Comportamento esperado da interface, resposta da API ou mensagem exibida]
