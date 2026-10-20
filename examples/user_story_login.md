# US01 - Recuperação de Senha por E-mail

**Como** usuário cadastrado na plataforma,
**Quero** solicitar a recuperação da minha senha através do meu e-mail,
**Para que** eu consiga redefinir meu acesso caso tenha esquecido minhas credenciais.

### Critérios de Aceitação:

1. A tela de login deve ter o link "Esqueci minha senha".
2. O usuário deve informar um e-mail válido.
3. Se o e-mail existir na base, enviar um link com token único válido por 15 minutos.
4. Se o e-mail não existir, exibir uma mensagem genérica por segurança: "Se o e-mail estiver cadastrado, você receberá as instruções em breve."
5. O formulário deve bloquear após 5 tentativas consecutivas com o mesmo IP (rate limit).
