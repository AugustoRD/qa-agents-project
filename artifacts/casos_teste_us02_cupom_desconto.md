# Especificação de Testes: US02 - Aplicação de Cupom de Desconto no Checkout

**Referência:** `examples/us02_cupom_desconto.md`

---

### Cenário: [US02-CN-01] - Aplicação de Cupom de Desconto Válido e Cálculo Sobre Itens Elegíveis

- **Categoria:** Caminho Feliz
- **Objetivo:** Validar a exibição do campo "Código do Cupom" e botão "Aplicar" no checkout e garantir que o desconto percentual seja abatido exclusivamente sobre o valor dos produtos elegíveis, sem aplicar desconto sobre o frete ou sobre itens não elegíveis.

##### Caso de Teste: [US02-CT-001] - Aplicação de cupom percentual válido em carrinho com produtos elegíveis

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Cliente autenticado com CPF "123.456.789-00" (sem utilização anterior do cupom); cupom "DESC10" válido (10% de desconto para pedidos acima de R$ 100,00); carrinho no checkout com 1 produto elegível de R$ 150,00 e frete de R$ 20,00 (Total inicial: R$ 170,00).
- **Passos de Execução (BDD):**
  **Dado** que o cliente está na etapa de checkout com um produto elegível no valor de "R$ 150,00" e frete de "R$ 20,00"
  **Quando** o cliente preenche o campo "Código do Cupom" com "DESC10"
  **E** aciona o botão "Aplicar"
  **Então** o sistema aplica o desconto de "R$ 15,00" apenas sobre o valor do produto elegível, mantém o frete em "R$ 20,00" e atualiza o valor total do pedido para "R$ 155,00"
- **Resultado Esperado Detalhado:** O resumo do pedido exibe claramente o valor dos produtos (R$ 150,00), o desconto concedido pelo cupom "DESC10" (-R$ 15,00), o valor integral do frete sem desconto (R$ 20,00) e o novo valor total a pagar (R$ 155,00).

##### Caso de Teste: [US02-CT-002] - Cálculo de desconto em carrinho misto (produtos elegíveis, não elegíveis e frete)

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Cliente autenticado sem uso prévio do cupom "DESC10" (10% de desconto, mínimo de R$ 100,00); carrinho contendo 1 produto elegível de R$ 120,00, 1 produto não elegível de R$ 80,00 e frete de R$ 25,00 (Subtotal de produtos: R$ 200,00; Total com frete: R$ 225,00).
- **Passos de Execução (BDD):**
  **Dado** que o cliente possui no checkout um produto elegível de "R$ 120,00", um produto não elegível de "R$ 80,00" e frete de "R$ 25,00"
  **Quando** o cliente insere o código "DESC10" no campo "Código do Cupom"
  **E** clica no botão "Aplicar"
  **Então** o sistema calcula os 10% de desconto exclusivamente sobre o produto elegível de "R$ 120,00", abatendo "R$ 12,00" e atualizando o total do pedido para "R$ 213,00"
- **Resultado Esperado Detalhado:** O produto não elegível (R$ 80,00) e o frete (R$ 25,00) permanecem com seus valores originais sem sofrer abatimento. O desconto exibido no resumo do pedido é de exatamente R$ 12,00 e o total final passa a ser R$ 213,00.

##### Caso de Teste: [US02-CT-003] - Aplicação de cupom digitado com letras minúsculas e espaços nas extremidades

- **Tipo:** Validação
- **Prioridade:** Baixa
- **Pré-condições:** Cliente no checkout com carrinho de R$ 150,00 em produtos elegíveis e frete de R$ 15,00; cupom "DESC10" válido e disponível.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com "R$ 150,00" em produtos elegíveis e "R$ 15,00" de frete
  **Quando** o cliente digita "  desc10  " no campo "Código do Cupom"
  **E** clica no botão "Aplicar"
  **Então** o sistema reconhece o código como "DESC10", aplica o desconto de "R$ 15,00" e atualiza o total do pedido para "R$ 150,00"
- **Resultado Esperado Detalhado:** O sistema desconsidera os espaços em branco antes e depois do código e a diferença entre letras maiúsculas e minúsculas, aplicando o benefício de 10% normalmente.

---

### Cenário: [US02-CN-02] - Validação de Valor Mínimo do Pedido (Análise de Valor Limite - BVA)

- **Categoria:** Caminho Alternativo
- **Objetivo:** Verificar a regra de valor mínimo de pedido exigida pelo cupom (R$ 100,00), garantindo o comportamento correto nos valores de fronteira e que o frete não seja somado para atingir esse valor mínimo.

##### Caso de Teste: [US02-CT-004] - Tentativa de aplicar cupom com valor de produtos abaixo do mínimo exigido (R$ 99,99)

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:** Cupom "DESC10" com exigência de valor mínimo de pedido de R$ 100,00; carrinho contendo R$ 99,99 em produtos elegíveis e R$ 20,00 de frete (Total geral: R$ 119,99).
- **Passos de Execução (BDD):**
  **Dado** que o cliente possui no carrinho "R$ 99,99" em produtos elegíveis e "R$ 20,00" de frete
  **Quando** o cliente preenche o campo "Código do Cupom" com "DESC10"
  **E** aciona o botão "Aplicar"
  **Então** o sistema impede a aplicação do cupom e informa ao cliente que o pedido não atingiu o valor mínimo de "R$ 100,00" para uso do cupom
- **Resultado Esperado Detalhado:** Nenhum desconto é aplicado ao pedido, o valor total permanece em R$ 119,99 e fica evidenciado que o valor do frete (R$ 20,00) não compõe o valor mínimo necessário para ativar o cupom.

##### Caso de Teste: [US02-CT-005] - Aplicação de cupom no limite exato do valor mínimo exigido (R$ 100,00)

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:** Cupom "DESC10" válido para pedidos a partir de R$ 100,00; carrinho contendo exatamente R$ 100,00 em produtos elegíveis e R$ 15,00 de frete.
- **Passos de Execução (BDD):**
  **Dado** que o cliente possui exatamente "R$ 100,00" em produtos elegíveis e "R$ 15,00" de frete no checkout
  **Quando** o cliente informa o cupom "DESC10"
  **E** clica no botão "Aplicar"
  **Então** o sistema aceita o cupom, concede "R$ 10,00" de desconto e atualiza o valor total do pedido para "R$ 105,00"
- **Resultado Esperado Detalhado:** O cupom "DESC10" é aceito no valor limite de R$ 100,00, exibindo o desconto de R$ 10,00 e o total atualizado de R$ 105,00 no resumo da compra.

##### Caso de Teste: [US02-CT-006] - Aplicação de cupom imediatamente acima do valor mínimo exigido (R$ 100,01)

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Cupom "DESC10" válido para compras acima de R$ 100,00; carrinho contendo R$ 100,01 em produtos elegíveis e R$ 15,00 de frete.
- **Passos de Execução (BDD):**
  **Dado** que o cliente possui "R$ 100,01" em produtos elegíveis e "R$ 15,00" de frete no checkout
  **Quando** o cliente insere o cupom "DESC10"
  **E** aciona o botão "Aplicar"
  **Então** o sistema aceita o cupom, aplica o desconto correspondente de "R$ 10,00" (arredondado em duas casas decimais) e atualiza o total do pedido para "R$ 105,01"
- **Resultado Esperado Detalhado:** O sistema valida que o pedido supera R$ 100,00, aplica os 10% sobre os produtos elegíveis mantendo a formatação monetária correta em reais e centavos (R$ 10,00 de desconto) e exibe o total de R$ 105,01.

---

### Cenário: [US02-CN-03] - Restrição de Múltiplos Cupons e Validação de Preenchimento do Campo

- **Categoria:** Caminho Alternativo
- **Objetivo:** Garantir que o cliente não consiga acumular mais de 1 cupom por pedido e validar o comportamento ao acionar o botão "Aplicar" sem informar um código.

##### Caso de Teste: [US02-CT-007] - Impedimento de aplicação de um segundo cupom no mesmo pedido

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:** Carrinho com R$ 200,00 em produtos elegíveis, frete de R$ 20,00 e o cupom "DESC10" já aplicado (desconto ativo de R$ 20,00; total atual de R$ 200,00); existência de outro cupom válido "PROMO5".
- **Passos de Execução (BDD):**
  **Dado** que o cliente já está com o cupom "DESC10" aplicado ao seu pedido no checkout
  **Quando** o cliente tenta inserir e aplicar um segundo cupom "PROMO5" na mesma compra
  **Então** o sistema impede a aplicação do segundo cupom e mantém apenas o desconto de "R$ 20,00" referente ao cupom "DESC10"
- **Resultado Esperado Detalhado:** O sistema não permite o acúmulo de dois cupons no mesmo pedido, bloqueando a adição de um novo código (ou informando que apenas 1 cupom é permitido por pedido) e preservando o valor total em R$ 200,00.

##### Caso de Teste: [US02-CT-008] - Tentativa de acionar o botão "Aplicar" com o campo "Código do Cupom" em branco

- **Tipo:** Validação
- **Prioridade:** Baixa
- **Pré-condições:** Cliente na tela de checkout com carrinho de R$ 150,00 e nenhum cupom aplicado.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com o campo "Código do Cupom" vazio
  **Quando** o cliente clica no botão "Aplicar"
  **Então** o sistema não aplica nenhum desconto e solicita o preenchimento de um código de cupom
- **Resultado Esperado Detalhado:** O valor total do pedido permanece inalterado e a tela orienta o usuário a digitar um código antes de aplicar.

---

### Cenário: [US02-CN-04] - Revogação Automática do Desconto ao Remover Produtos do Carrinho

- **Categoria:** Caminho Alternativo
- **Objetivo:** Validar que o desconto do cupom é revogado automaticamente caso o cliente remova produtos do carrinho e o valor passe a ficar abaixo do mínimo exigido pelo cupom.

##### Caso de Teste: [US02-CT-009] - Revogação automática do desconto quando a remoção de produto deixa o total abaixo do valor mínimo

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Carrinho contendo "Produto A (R$ 70,00)" e "Produto B (R$ 50,00)", totalizando R$ 120,00 em produtos elegíveis, frete de R$ 15,00 e cupom "DESC10" (mínimo R$ 100,00) já aplicado com desconto de R$ 12,00 (Total com desconto: R$ 123,00).
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com o cupom "DESC10" aplicado sobre "Produto A (R$ 70,00)" e "Produto B (R$ 50,00)"
  **Quando** o cliente remove o "Produto B (R$ 50,00)" do carrinho, reduzindo o valor dos produtos para "R$ 70,00"
  **Então** o sistema revoga automaticamente o cupom "DESC10", zera o valor de desconto e recalcula o total do pedido para "R$ 85,00"
- **Resultado Esperado Detalhado:** Assim que o valor dos produtos cai para R$ 70,00 (abaixo do mínimo de R$ 100,00), o cupom "DESC10" é removido automaticamente do resumo da compra, o desconto passa a ser R$ 0,00 e o total a pagar reflete apenas o Produto A mais o frete (R$ 70,00 + R$ 15,00 = R$ 85,00).

##### Caso de Teste: [US02-CT-010] - Recálculo e manutenção do cupom quando a remoção de um item mantém o carrinho acima do valor mínimo

- **Tipo:** Funcional
- **Prioridade:** Média
- **Pré-condições:** Carrinho contendo "Produto A (R$ 120,00)" e "Produto B (R$ 50,00)", totalizando R$ 170,00 em produtos elegíveis, frete de R$ 20,00 e cupom "DESC10" aplicado (desconto inicial de R$ 17,00).
- **Passos de Execução (BDD):**
  **Dado** que o cliente possui no carrinho "Produto A (R$ 120,00)" e "Produto B (R$ 50,00)" com o cupom "DESC10" aplicado
  **Quando** o cliente remove o "Produto B (R$ 50,00)", mantendo "R$ 120,00" em produtos elegíveis
  **Então** o sistema mantém o cupom "DESC10" ativo, ajusta o desconto proporcionalmente para "R$ 12,00" e atualiza o total do pedido para "R$ 128,00"
- **Resultado Esperado Detalhado:** Como o valor restante de produtos (R$ 120,00) continua acima do mínimo de R$ 100,00, o cupom não é revogado, mas seu desconto é recalculado para 10% do novo valor (R$ 12,00), somando R$ 128,00 com o frete de R$ 20,00.

---

### Cenário: [US02-CN-05] - Tratamento de Cupons Expirados, Esgotados e Inexistentes

- **Categoria:** Exceção e Segurança
- **Objetivo:** Validar a exibição das mensagens de erro exatas definidas nos critérios de aceitação para cupons fora da validade, com limite de utilizações atingido ou códigos inválidos.

##### Caso de Teste: [US02-CT-011] - Tentativa de aplicar um cupom com prazo de validade expirado

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:** Carrinho no checkout com R$ 150,00 em produtos elegíveis; cupom "NATAL10" cadastrado porém com data de validade vencida.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com "R$ 150,00" em produtos elegíveis
  **Quando** o cliente insere o código vencido "NATAL10" no campo "Código do Cupom"
  **E** clica no botão "Aplicar"
  **Então** o sistema não concede desconto e exibe exatamente a mensagem "Cupom expirado."
- **Resultado Esperado Detalhado:** A tela de checkout apresenta a mensagem de erro `"Cupom expirado."` e mantém o valor total do pedido sem nenhuma redução.

##### Caso de Teste: [US02-CT-012] - Tentativa de aplicar um cupom com limite de utilizações esgotado

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:** Carrinho no checkout com R$ 150,00 em produtos elegíveis; cupom "FLASH10" dentro do prazo de validade, mas com sua quantidade máxima de usos já atingida.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com "R$ 150,00" em produtos elegíveis
  **Quando** o cliente informa o cupom esgotado "FLASH10" no campo "Código do Cupom"
  **E** aciona o botão "Aplicar"
  **Então** o sistema recusa a aplicação e exibe exatamente a mensagem "Limite de utilizações deste cupom atingido."
- **Resultado Esperado Detalhado:** O checkout exibe ao cliente a mensagem `"Limite de utilizações deste cupom atingido."` e preserva o valor original do pedido.

##### Caso de Teste: [US02-CT-013] - Tentativa de aplicar um código de cupom inexistente

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:** Carrinho no checkout com R$ 150,00 em produtos elegíveis; código "CUPOM999" não cadastrado como promoção na loja.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está no checkout com "R$ 150,00" em produtos elegíveis
  **Quando** o cliente digita o código inexistente "CUPOM999" no campo "Código do Cupom"
  **E** clica no botão "Aplicar"
  **Então** o sistema rejeita o código e exibe exatamente a mensagem "Cupom inválido."
- **Resultado Esperado Detalhado:** O sistema exibe a mensagem `"Cupom inválido."` na tela e não altera os valores do resumo da compra.

##### Caso de Teste: [US02-CT-014] - Tentativa de aplicar código preenchido com caracteres especiais inválidos

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Cliente no checkout com carrinho de R$ 150,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  **Dado** que o cliente está na etapa de checkout com um carrinho de "R$ 150,00"
  **Quando** o cliente insere o texto com símbolos especiais "DESC@#10$%" no campo "Código do Cupom"
  **E** clica no botão "Aplicar"
  **Então** o sistema rejeita a entrada e exibe a mensagem "Cupom inválido."
- **Resultado Esperado Detalhado:** O sistema trata os símbolos especiais sem falhas na tela, não aplica nenhum desconto e exibe a mensagem padrão `"Cupom inválido."`.

---

### Cenário: [US02-CN-06] - Restrição de Uso Único por CPF do Cliente

- **Categoria:** Exceção e Segurança
- **Objetivo:** Garantir que cada cupom promocional possa ser utilizado apenas uma única vez pelo mesmo CPF, impedindo a reutilização em novos pedidos ou em sessões simultâneas.

##### Caso de Teste: [US02-CT-015] - Bloqueio de reutilização de cupom em uma nova compra pelo mesmo CPF

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Cliente portador do CPF "123.456.789-00" que já concluiu um pedido anterior utilizando o cupom "DESC10"; novo carrinho no checkout com R$ 200,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  **Dado** que o cliente com CPF "123.456.789-00" já utilizou o cupom "DESC10" em uma compra anterior
  **Quando** o cliente informa novamente o código "DESC10" em um novo pedido de "R$ 200,00"
  **E** aciona o botão "Aplicar"
  **Então** o sistema impede a aplicação do desconto por já constar utilização prévia do cupom para este CPF
- **Resultado Esperado Detalhado:** O sistema identifica que o CPF "123.456.789-00" já usufruiu do cupom "DESC10", informa ao cliente que o cupom é de uso único por CPF e mantém o valor total do pedido sem desconto.

##### Caso de Teste: [US02-CT-016] - Bloqueio de uso duplicado do mesmo cupom pelo mesmo CPF em duas compras abertas simultaneamente

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Cliente portador do CPF "123.456.789-00" (sem uso prévio do cupom "DESC10") com duas sessões de checkout abertas ao mesmo tempo, ambas com R$ 150,00 em produtos elegíveis.
- **Passos de Execução (BDD):**
  **Dado** que o cliente com CPF "123.456.789-00" aplicou o cupom "DESC10" em duas sessões de checkout abertas paralelamente
  **Quando** o cliente finaliza o primeiro pedido utilizando o cupom "DESC10" e em seguida tenta concluir o segundo pedido com o mesmo cupom
  **Então** o sistema conclui apenas o primeiro pedido com desconto e bloqueia o uso do cupom "DESC10" no segundo pedido
- **Resultado Esperado Detalhado:** O sistema garante que o limite de 1 uso por CPF não seja contornado por abas simultâneas, revogando o desconto no segundo checkout e informando que o cupom já foi utilizado por este CPF.
