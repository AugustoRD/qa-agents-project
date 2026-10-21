# US02 - Aplicação de Cupom de Desconto no Checkout

**Como** cliente final na etapa de finalização de compra,  
**Quero** inserir um cupom de desconto promocional no carrinho,  
**Para que** eu obtenha uma redução no valor total do meu pedido antes de realizar o pagamento.

### Critérios de Aceitação:

1. O checkout deve exibir um campo de texto "Código do Cupom" com um botão "Aplicar".
2. Cupons válidos de porcentagem (ex: `DESC10` para 10%) devem abater o valor exclusivamente sobre os produtos elegíveis (o frete não recebe desconto).
3. O cupom deve ter validação de valor mínimo de pedido (ex: compras acima de R$ 100,00).
4. O sistema deve impedir a aplicação de múltiplos cupons na mesma compra (apenas 1 cupom por pedido).
5. Se o cupom estiver expirado, com limite de usos esgotado ou for inexistente, o sistema deve exibir mensagens claras de erro:
   - "Cupom expirado."
   - "Limite de utilizações deste cupom atingido."
   - "Cupom inválido."
6. O cupom deve ser de uso único por CPF do cliente.
7. Se o usuário remover produtos do carrinho e o total ficar abaixo do valor mínimo do cupom, o desconto deve ser revogado automaticamente.
