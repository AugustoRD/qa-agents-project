# Especificação de Testes: US02 - Aplicação de Cupom de Desconto no Checkout

**Referência:** `examples/us02_cupom_desconto.md`

---

### Cenário: [CN-01] - Aplicação de Cupom de Desconto com Sucesso

- **Categoria:** Caminho Feliz
- **Objetivo:** Validar a aplicação correta de um cupom promocional válido no checkout, garantindo que o desconto percentual incida exclusivamente sobre os produtos elegíveis sem alterar o valor do frete.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-001] - Aplicar cupom percentual válido sobre produtos elegíveis sem abater o frete

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente autenticado com CPF que nunca utilizou o cupom `DESC10`.
  - Cupom `DESC10` (10% de desconto, valor mínimo de R$ 100,00) ativo e dentro do limite de usos.
  - Carrinho de compras contendo R$ 150,00 em produtos elegíveis e R$ 20,00 de frete (Total inicial: R$ 170,00).
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está na tela de checkout com R$ 150,00 em produtos elegíveis e R$ 20,00 de frete
  - **Quando** o cliente insere o código `DESC10` no campo "Código do Cupom"
  - **E** clica no botão "Aplicar"
  - **Então** o sistema deve aplicar um desconto de R$ 15,00 (10% sobre R$ 150,00)
  - **E** manter o valor do frete inalterado em R$ 20,00, atualizando o valor total do pedido para R$ 155,00.
- **Resultado Esperado Detalhado:** A interface exibe a confirmação do cupom `DESC10` aplicado, discrimina o desconto de "- R$ 15,00" no resumo do pedido, mantém o frete em "R$ 20,00" e exibe o total atualizado de "R$ 155,00".

##### Caso de Teste: [CT-002] - Aplicar cupom em carrinho misto (produtos elegíveis e não elegíveis)

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente autenticado com CPF apto a usar o cupom `DESC10`.
  - Cupom `DESC10` (10% de desconto, valor mínimo R$ 100,00) ativo.
  - Carrinho contendo R$ 120,00 em produtos elegíveis, R$ 80,00 em produtos não elegíveis e R$ 25,00 de frete.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está no checkout com R$ 120,00 em produtos elegíveis e R$ 80,00 em produtos não elegíveis
  - **Quando** o cliente insere o cupom `DESC10` no campo "Código do Cupom"
  - **E** aciona o botão "Aplicar"
  - **Então** o sistema deve calcular os 10% exclusivamente sobre os R$ 120,00 de produtos elegíveis (desconto de R$ 12,00)
  - **E** não deve aplicar desconto sobre os produtos não elegíveis nem sobre o frete.
- **Resultado Esperado Detalhado:** O resumo do pedido apresenta um desconto exato de R$ 12,00, totalizando R$ 213,00 (R$ 108,00 elegíveis com desconto + R$ 80,00 não elegíveis + R$ 25,00 frete).

---

### Cenário: [CN-02] - Validação de Regras de Negócio e Limites de Valor Mínimo

- **Categoria:** Caminho Alternativo
- **Objetivo:** Verificar o comportamento do sistema diante das regras de valor mínimo de pedido (análise de valor limite), tentativa de acúmulo de cupons e revogação dinâmica ao remover itens do carrinho.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-003] - Impedir aplicação de cupom quando o subtotal elegível for inferior ao valor mínimo (Valor Limite Inferior)

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:**
  - Cupom `DESC10` configurado com valor mínimo de pedido de R$ 100,00.
  - Carrinho contendo R$ 99,99 em produtos elegíveis e R$ 20,00 de frete (Total com frete: R$ 119,99).
- **Passos de Execução (BDD):**
  - **Dado** que o cliente possui R$ 99,99 em produtos no carrinho e R$ 20,00 de frete
  - **Quando** o cliente informa o código `DESC10` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema deve recusar a aplicação do cupom por não atingir o valor mínimo de R$ 100,00 em produtos.
- **Resultado Esperado Detalhado:** O cupom não é aplicado (o frete não compõe o valor mínimo do pedido), o valor total permanece R$ 119,99 e o sistema exibe mensagem informativa indicando que o valor mínimo de R$ 100,00 não foi atingido.

##### Caso de Teste: [CT-004] - Permitir aplicação de cupom no limite exato do valor mínimo (Valor Limite Exato)

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:**
  - Cupom `DESC10` configurado com valor mínimo de R$ 100,00 (acima ou igual a R$ 100,00) ou R$ 100,01 (estritamente acima de R$ 100,00, conforme regra parametrizada).
  - Carrinho com subtotal de produtos elegíveis no limite exato exigido pela regra (R$ 100,00 / R$ 100,01).
- **Passos de Execução (BDD):**
  - **Dado** que o cliente possui o valor limite exato exigido em produtos elegíveis no checkout
  - **Quando** o cliente insere `DESC10` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema deve validar o limite com sucesso e aplicar os 10% de desconto.
- **Resultado Esperado Detalhado:** O desconto é concedido sem erros de arredondamento de ponto flutuante e o resumo de valores é atualizado na tela.

##### Caso de Teste: [CT-005] - Impedir aplicação de múltiplos cupons no mesmo pedido

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:**
  - Cupons `DESC10` e `PROMO20` ativos e válidos.
  - Cliente no checkout com carrinho de R$ 200,00 e cupom `DESC10` já aplicado.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente já aplicou o cupom `DESC10` ao seu pedido no checkout
  - **Quando** o cliente tenta inserir e aplicar um segundo cupom `PROMO20`
  - **Então** o sistema deve impedir o acúmulo dos dois cupons no mesmo pedido.
- **Resultado Esperado Detalhado:** O sistema bloqueia a adição simultânea (desabilitando o campo após 1 cupom aplicado ou exibindo alerta de que apenas 1 cupom é permitido por compra), garantindo que nunca haja desconto cumulativo.

##### Caso de Teste: [CT-006] - Revogar desconto automaticamente ao remover produtos e ficar abaixo do valor mínimo

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:**
  - Cupom `DESC10` (valor mínimo R$ 100,00) aplicado em um carrinho com 2 produtos de R$ 60,00 cada (Subtotal: R$ 120,00; Desconto: R$ 12,00).
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está com o cupom `DESC10` aplicado em um carrinho de R$ 120,00
  - **Quando** o cliente remove um dos produtos de R$ 60,00 do carrinho
  - **E** o novo subtotal de produtos passa a ser R$ 60,00 (abaixo do mínimo de R$ 100,00)
  - **Então** o sistema deve revogar automaticamente o cupom `DESC10` e remover o desconto do pedido.
- **Resultado Esperado Detalhado:** O desconto é removido imediatamente sem necessidade de recarregar a página manualmente, o subtotal/total passa a refletir R$ 60,00 + frete (sem os 10% de abatimento) e o usuário é notificado da remoção do cupom por não atender mais ao valor mínimo.

---

### Cenário: [CN-03] - Tratamento de Cupons Inválidos, Expirados, Esgotados e Uso Único por CPF

- **Categoria:** Exceção e Segurança
- **Objetivo:** Garantir que cupons fora de vigência, com limite global esgotado, inexistentes ou já utilizados pelo mesmo CPF sejam rejeitados com as mensagens exatas especificadas.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-007] - Rejeitar aplicação de cupom com data de validade expirada

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:**
  - Cupom `VERAO10` cadastrado na base, porém com data/hora de expiração vencida.
  - Carrinho com R$ 150,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está no checkout com um pedido de R$ 150,00
  - **Quando** o cliente insere o cupom vencido `VERAO10` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema não deve conceder desconto e deve exibir a mensagem de erro "Cupom expirado.".
- **Resultado Esperado Detalhado:** A API retorna status de erro de negócio (ex: 400/422), o valor do pedido permanece inalterado e a interface exibe exatamente o texto `"Cupom expirado."`.

##### Caso de Teste: [CT-008] - Rejeitar aplicação de cupom com limite de utilizações esgotado

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:**
  - Cupom `BLACK50` vigente, mas com limite máximo de usos globais já atingido (ex: 100/100 usos concluídos).
  - Carrinho com R$ 150,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está no checkout com um pedido de R$ 150,00
  - **Quando** o cliente insere o cupom `BLACK50` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema deve rejeitar o cupom e exibir a mensagem "Limite de utilizações deste cupom atingido.".
- **Resultado Esperado Detalhado:** Nenhum desconto é aplicado ao carrinho e a interface apresenta exatamente a mensagem `"Limite de utilizações deste cupom atingido."`.

##### Caso de Teste: [CT-009] - Rejeitar aplicação de cupom inexistente ou com código em branco

- **Tipo:** Exceção
- **Prioridade:** Média
- **Pré-condições:**
  - Código `CUPOMFAKE999` não existente na base de dados.
  - Carrinho com R$ 150,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está no checkout com um pedido de R$ 150,00
  - **Quando** o cliente digita o código inexistente `CUPOMFAKE999` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema deve recusar a operação e exibir a mensagem "Cupom inválido.".
- **Resultado Esperado Detalhado:** O total do carrinho não sofre alterações e a interface exibe exatamente a mensagem `"Cupom inválido."`.

##### Caso de Teste: [CT-010] - Impedir reutilização do mesmo cupom pelo mesmo CPF (inclusive em outra conta)

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente com CPF `123.456.789-00` já finalizou um pedido anterior utilizando o cupom `DESC10`.
  - Cupom `DESC10` continua ativo e com usos globais disponíveis.
- **Passos de Execução (BDD):**
  - **Dado** que o cliente autenticado possui o CPF `123.456.789-00` que já consumiu o cupom `DESC10` anteriormente
  - **Quando** o cliente tenta aplicar novamente o cupom `DESC10` em um novo checkout de R$ 200,00
  - **Então** o sistema deve bloquear o uso do cupom para este CPF.
- **Resultado Esperado Detalhado:** O sistema valida o histórico de uso vinculado ao CPF (independentemente de e-mail ou sessão), nega o desconto e exibe mensagem clara informando que o cupom já foi utilizado por este CPF.

---

### Cenário: [CN-04] - Segurança, Concorrência e Prevenção a Abusos no Checkout

- **Categoria:** Exceção e Segurança
- **Objetivo:** Assegurar que o endpoint e a interface de cupons sejam resilientes contra força bruta (enumeração de cupons), condições de corrida (race conditions), injeção de código e manipulação de parâmetros no cliente.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-011] - Bloquear força bruta e enumeração de cupons via Rate Limiting

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente no checkout realizando tentativas sequenciais automatizadas no endpoint de validação de cupom.
- **Passos de Execução (BDD):**
  - **Dado** que um usuário ou script envia múltiplas requisições seguidas com códigos aleatórios no campo "Código do Cupom" (ex: mais de 10 tentativas em 1 minuto)
  - **Quando** o limite de requisições por IP/sessão é ultrapassado
  - **Então** o sistema deve bloquear temporariamente novas validações (HTTP 429 - Too Many Requests).
- **Resultado Esperado Detalhado:** O servidor aplica rate limiting para impedir descoberta de cupons por força bruta e retorna mensagem amigável sem expor detalhes internos da infraestrutura.

##### Caso de Teste: [CT-012] - Garantir integridade em condição de corrida (Race Condition) para uso único por CPF ou último uso global

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente com o mesmo CPF aberto em duas abas/dispositivos simultâneos no checkout, ou dois clientes disputando a última unidade disponível de um cupom.
- **Passos de Execução (BDD):**
  - **Dado** que o cupom `DESC10` possui apenas 1 uso restante (para o CPF ou no limite global)
  - **Quando** duas requisições concorrentes de aplicação/finalização de pedido com o cupom `DESC10` são disparadas exatamente ao mesmo tempo
  - **Então** o sistema deve processar apenas uma transação com sucesso e rejeitar a segunda por bloqueio transacional/atômico.
- **Resultado Esperado Detalhado:** O banco de dados mantém consistência atômica (lock pessimista/otimista), impedindo que o limite de 1 uso por CPF ou limite máximo do cupom seja ultrapassado.

##### Caso de Teste: [CT-013] - Prevenir Injeção (SQLi / XSS) e vazamento de dados no campo "Código do Cupom"

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:**
  - Cliente na tela de checkout com acesso ao campo "Código do Cupom".
- **Passos de Execução (BDD):**
  - **Dado** que o cliente está na etapa de checkout
  - **Quando** o cliente insere payloads maliciosos como `' OR 1=1 --` ou `<script>alert(1)</script>` no campo "Código do Cupom"
  - **E** clica em "Aplicar"
  - **Então** o sistema deve sanitizar/parametrizar a entrada, não executar scripts e retornar apenas "Cupom inválido.".
- **Resultado Esperado Detalhado:** Nenhum comando SQL ou script JS é interpretado; a resposta da API não vaza stack traces, estrutura de tabelas ou regras internas de outros cupons, retornando estritamente `"Cupom inválido."`.
