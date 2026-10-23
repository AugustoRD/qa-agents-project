# Especificação de Testes: US01 - Recuperação de Senha por E-mail

**Referência:** `examples/user_story_login.md`

---

### Cenário: [US01-CN-01] - Solicitação e Redefinição de Senha com Sucesso

- **Categoria:** Caminho Feliz
- **Objetivo:** Validar o acesso à tela de recuperação a partir do login, a geração e envio do link com token único para e-mail cadastrado e a utilização do token dentro do prazo de validade de 15 minutos.

##### Caso de Teste: [US01-CT-001] - Navegar para a tela de recuperação de senha através do link na tela de login

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Usuário não autenticado acessando a tela de login da plataforma (`/login`).
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de login da plataforma
  **Quando** visualiza e clica no link `"Esqueci minha senha"`
  **Então** o sistema redireciona o usuário para a tela de solicitação de recuperação de senha contendo o campo de e-mail e o botão de envio
- **Resultado Esperado Detalhado:** O link `"Esqueci minha senha"` deve estar visível e acessível na interface de login. Ao clicar, a rota deve mudar para `/recuperar-senha` (HTTP 200), exibindo o formulário de recuperação sem erros de carregamento.

##### Caso de Teste: [US01-CT-002] - Solicitar recuperação de senha informando um e-mail válido e cadastrado

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Usuário cadastrado e ativo na base de dados com o e-mail `"usuario.cadastrado@dominio.com"`; nenhuma tentativa prévia de recuperação bloqueada para o IP atual.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** preenche o campo de e-mail com `"usuario.cadastrado@dominio.com"`
  **E** aciona o botão de enviar solicitação
  **Então** o sistema exibe a mensagem `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."` e envia um e-mail contendo um link com token único válido por `"15 minutos"`
- **Resultado Esperado Detalhado:** A API deve retornar status HTTP 200 OK. Um registro de token criptograficamente seguro, único e vinculado ao usuário deve ser persistido no banco com TTL/expiração de exatos 15 minutos a partir do timestamp de geração, e o serviço de mensageria deve disparar o e-mail com o link de redefinição.

##### Caso de Teste: [US01-CT-003] - Acessar o link de recuperação no limite exato de validade do token (15 minutos)

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Token único `"tok_valido_987654"` gerado há exatos `"15 minutos"` (900 segundos) para o e-mail `"usuario.cadastrado@dominio.com"` e ainda não utilizado.
- **Passos de Execução (BDD):**
  **Dado** que o usuário recebeu o link de recuperação com o token `"tok_valido_987654"` gerado há exatos `"15 minutos"`
  **Quando** acessa a URL de redefinição contendo o token `"tok_valido_987654"` no minuto `"15:00"`
  **Então** o sistema valida o token com sucesso e permite a redefinição da senha
- **Resultado Esperado Detalhado:** Análise de Valor Limite (BVA) no limite superior válido (`t = 15m00s`). A API valida o token (HTTP 200), autoriza a exibição/submissão do formulário de nova senha e invalida o token imediatamente após a conclusão da redefinição.

---

### Cenário: [US01-CN-02] - Validações de Formato, Normalização e Limites de Tentativas Válidas

- **Categoria:** Caminho Alternativo
- **Objetivo:** Validar o tratamento de entradas inválidas ou vazias no campo de e-mail (Particionamento de Equivalência), a normalização de dados (espaços e letras maiúsculas) e o comportamento no limite exato permitido antes do bloqueio de Rate Limit.

##### Caso de Teste: [US01-CT-004] - Solicitar recuperação informando e-mail com letras maiúsculas e espaços nas extremidades

- **Tipo:** Validação
- **Prioridade:** Baixa
- **Pré-condições:** Usuário cadastrado na base como `"usuario.cadastrado@dominio.com"`.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** informa o e-mail `"  USUARIO.CADASTRADO@DOMINIO.COM  "` com espaços em branco no início e no fim e caracteres em caixa alta
  **E** submete o formulário de recuperação
  **Então** o sistema aplica normalização (`trim` e `lowercase`), reconhece o cadastro e envia o link com token único normalmente
- **Resultado Esperado Detalhado:** O front-end/back-end deve higienizar a string para `"usuario.cadastrado@dominio.com"`, retornar HTTP 200 OK com a mensagem padrão de confirmação e disparar o e-mail de recuperação para o usuário.

##### Caso de Teste: [US01-CT-005] - Tentar solicitar recuperação com o campo de e-mail vazio

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Usuário na tela de recuperação de senha.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** mantém o campo de e-mail vazio `""`
  **E** clica no botão de enviar solicitação
  **Então** o sistema impede o envio da requisição e exibe uma mensagem de validação informando que o campo de e-mail é obrigatório
- **Resultado Esperado Detalhado:** O formulário deve destacar o campo de e-mail com estado de erro visual e mensagem clara de obrigatoriedade. Caso a validação ocorra na API, deve retornar status HTTP 400 Bad Request ou 422 Unprocessable Entity sem consumir a fila de envio de e-mails.

##### Caso de Teste: [US01-CT-006] - Tentar solicitar recuperação informando e-mail com formato inválido

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Usuário na tela de recuperação de senha.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** preenche o campo de e-mail com um formato inválido como `"usuario.dominio.com"` (sem `@`) ou `"usuario@"` (sem domínio)
  **E** clica no botão de enviar solicitação
  **Então** o sistema rejeita a entrada e exibe uma mensagem solicitando um endereço de e-mail válido
- **Resultado Esperado Detalhado:** Particionamento de Equivalência para classe de formato inválido. O sistema não deve disparar busca no banco de dados nem envio de e-mail, retornando erro de validação na interface (e HTTP 400/422 se submetido via API).

##### Caso de Teste: [US01-CT-007] - Realizar a 5ª tentativa consecutiva de recuperação a partir do mesmo IP (Limite Superior Válido)

- **Tipo:** Validação
- **Prioridade:** Alta
- **Pré-condições:** O endereço IP `"192.168.1.50"` já realizou exatamente `"4"` tentativas consecutivas de recuperação de senha dentro da janela de rate limit.
- **Passos de Execução (BDD):**
  **Dado** que o usuário originado do IP `"192.168.1.50"` já realizou `"4"` solicitações consecutivas de recuperação
  **Quando** submete a `"5ª"` solicitação consecutiva com o e-mail `"usuario.cadastrado@dominio.com"`
  **Então** o sistema processa a `"5ª"` tentativa com sucesso e exibe `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."`
- **Resultado Esperado Detalhado:** Análise de Valor Limite (BVA) para a regra "bloquear após 5 tentativas consecutivas". A 5ª tentativa está dentro do limite permitido (`tentativas <= 5`), devendo retornar HTTP 200 OK e registrar o contador do IP `"192.168.1.50"` em `5`.

##### Caso de Teste: [US01-CT-008] - Solicitar novo link de recuperação quando já existe um token ativo para o mesmo e-mail

- **Tipo:** Funcional
- **Prioridade:** Média
- **Pré-condições:** Usuário `"usuario.cadastrado@dominio.com"` possui um token `"tok_anterior_111"` gerado há `"3 minutos"` ainda não utilizado.
- **Passos de Execução (BDD):**
  **Dado** que o usuário `"usuario.cadastrado@dominio.com"` possui um token ativo `"tok_anterior_111"` emitido há `"3 minutos"`
  **Quando** solicita novamente a recuperação de senha informando `"usuario.cadastrado@dominio.com"`
  **Então** o sistema gera um novo token único `"tok_novo_222"` válido por `"15 minutos"` e revoga imediatamente o token anterior `"tok_anterior_111"`
- **Resultado Esperado Detalhado:** Transição de estado do ciclo de vida do token. Apenas o último token emitido deve permanecer válido no banco de dados, evitando múltiplos links ativos simultaneamente para a mesma conta.

---

### Cenário: [US01-CN-03] - Prevenção à Enumeração de Usuários e Ciclo de Vida de Expiração/Uso do Token

- **Categoria:** Exceção e Segurança
- **Objetivo:** Garantir que e-mails não cadastrados recebam resposta genérica indistinguível (OWASP User Enumeration), e validar que tokens expirados (> 15 min) ou já utilizados sejam rejeitados.

##### Caso de Teste: [US01-CT-009] - Solicitar recuperação de senha informando um e-mail não cadastrado na base

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** O e-mail `"nao.existe@dominio.com"` não possui cadastro na plataforma.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** preenche o campo de e-mail com `"nao.existe@dominio.com"`
  **E** aciona o botão de enviar solicitação
  **Então** o sistema exibe exatamente a mensagem genérica `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."` sem enviar nenhum e-mail
- **Resultado Esperado Detalhado:** Prevenção a enumeração de usuários (OWASP). O status HTTP retornado (HTTP 200 OK), a estrutura do payload de resposta, o layout da tela e o tempo médio de resposta devem ser idênticos aos de um e-mail existente, sem gerar token no banco nem disparar e-mail.

##### Caso de Teste: [US01-CT-010] - Tentar utilizar o link de recuperação após o prazo limite de 15 minutos (Token Expirado)

- **Tipo:** Exceção
- **Prioridade:** Alta
- **Pré-condições:** Token único `"tok_expirado_1501"` gerado há `"15 minutos e 1 segundo"` (901 segundos) para `"usuario.cadastrado@dominio.com"`.
- **Passos de Execução (BDD):**
  **Dado** que o token `"tok_expirado_1501"` foi gerado há `"15 minutos e 1 segundo"`
  **Quando** o usuário acessa o link de recuperação contendo o token `"tok_expirado_1501"`
  **Então** o sistema bloqueia a redefinição de senha e informa que o link expirou, orientando o usuário a realizar uma nova solicitação
- **Resultado Esperado Detalhado:** Análise de Valor Limite (BVA) imediatamente acima do limite (`t = 15m01s`). A API deve rejeitar o token com HTTP 400 Bad Request, 401 Unauthorized ou 410 Gone, impedindo a alteração da senha e mantendo a credencial atual intacta.

##### Caso de Teste: [US01-CT-011] - Tentar reutilizar um token único que já foi consumido anteriormente

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Token `"tok_usado_777"` emitido há `"5 minutos"` e já utilizado com sucesso para redefinir a senha de `"usuario.cadastrado@dominio.com"`.
- **Passos de Execução (BDD):**
  **Dado** que o usuário já redefiniu sua senha utilizando o token `"tok_usado_777"` há `"5 minutos"`
  **Quando** tenta acessar ou submeter uma nova redefinição utilizando o mesmo link com o token `"tok_usado_777"` ainda dentro da janela de `"15 minutos"`
  **Então** o sistema rejeita a requisição e informa que o link de recuperação é inválido ou já foi utilizado
- **Resultado Esperado Detalhado:** Validação de uso único (Single-Use Token). Após o primeiro consumo, o status do token no banco deve transitar para `UTILIZADO`/`INVALIDADO`, retornando HTTP 400/401/410 em qualquer tentativa subsequente.

---

### Cenário: [US01-CN-04] - Bloqueio por Rate Limit (Força Bruta), Concorrência e Ataques de Injeção

- **Categoria:** Exceção e Segurança
- **Objetivo:** Validar o bloqueio do formulário após 5 tentativas consecutivas do mesmo IP, a proteção contra burla de cabeçalhos, condições de corrida no uso do token e injeção de código malicioso.

##### Caso de Teste: [US01-CT-012] - Bloquear o formulário na 6ª tentativa consecutiva de recuperação a partir do mesmo IP (Rate Limit)

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** O endereço IP `"192.168.1.50"` acabou de realizar `"5"` tentativas consecutivas de recuperação de senha.
- **Passos de Execução (BDD):**
  **Dado** que o endereço IP `"192.168.1.50"` já atingiu `"5"` tentativas consecutivas no formulário de recuperação
  **Quando** o usuário realiza a `"6ª"` tentativa consecutiva a partir do mesmo IP `"192.168.1.50"` informando `"usuario.cadastrado@dominio.com"`
  **Então** o sistema bloqueia a submissão do formulário e rejeita a requisição por excesso de tentativas
- **Resultado Esperado Detalhado:** Análise de Valor Limite (BVA) no primeiro valor inválido (`tentativas = 6`). A interface deve desabilitar/bloquear a submissão e exibir alerta de limite de tentativas excedido; a API deve retornar status `HTTP 429 Too Many Requests` (com cabeçalho `Retry-After`) sem processar busca no banco nem enviar e-mail.

##### Caso de Teste: [US01-CT-013] - Tentar burlar o bloqueio de Rate Limit falsificando o cabeçalho X-Forwarded-For após 5 tentativas

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** O endereço IP real do cliente `"192.168.1.50"` encontra-se bloqueado após `"5"` tentativas consecutivas.
- **Passos de Execução (BDD):**
  **Dado** que o IP `"192.168.1.50"` está bloqueado pelo limite de `"5"` tentativas consecutivas
  **Quando** um atacante envia a `"6ª"` requisição a partir do IP `"192.168.1.50"` injetando o cabeçalho HTTP `"X-Forwarded-For: 10.0.0.99"`
  **Então** o sistema ignora cabeçalhos de proxy não confiáveis e mantém o bloqueio da requisição com erro de limite excedido
- **Resultado Esperado Detalhado:** A identificação de IP para rate limiting deve utilizar o IP real da conexão ou apenas proxies reversos confiáveis configurados na infraestrutura, retornando `HTTP 429 Too Many Requests`.

##### Caso de Teste: [US01-CT-014] - Disparar requisições simultâneas (Race Condition) utilizando o mesmo token único de recuperação

- **Tipo:** Segurança
- **Prioridade:** Média
- **Pré-condições:** Token único `"tok_concorrente_888"` válido e ativo para `"usuario.cadastrado@dominio.com"`.
- **Passos de Execução (BDD):**
  **Dado** que o token `"tok_concorrente_888"` está ativo e dentro do prazo de `"15 minutos"`
  **Quando** duas requisições paralelas e simultâneas de redefinição de senha são enviadas no mesmo milissegundo utilizando o token `"tok_concorrente_888"`
  **Então** o sistema processa apenas uma das requisições com sucesso e rejeita a segunda requisição por conflito ou token já utilizado
- **Resultado Esperado Detalhado:** Controle de concorrência e atomicidade no banco de dados (lock pessimista/otimista ou `UPDATE ... WHERE used = false`). Apenas uma requisição retorna HTTP 200 OK e a outra deve receber HTTP 400/409 Conflict, garantindo uso estritamente único.

##### Caso de Teste: [US01-CT-015] - Tentar submeter payloads de SQL Injection e XSS no campo de e-mail

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Usuário ou atacante na tela/endpoint de recuperação de senha.
- **Passos de Execução (BDD):**
  **Dado** que o usuário está na tela de recuperação de senha
  **Quando** submete no campo de e-mail o payload malicioso `"' OR 1=1 -- <script>alert('xss')</script>@dominio.com"`
  **Então** o sistema sanitiza e rejeita a entrada sem executar scripts no navegador e sem expor erros internos de banco de dados
- **Resultado Esperado Detalhado:** Proteção OWASP Injection/XSS. A aplicação deve rejeitar o valor como formato de e-mail inválido (HTTP 400/422) ou tratá-lo com queries parametrizadas, jamais retornando `HTTP 500 Internal Server Error`, stack traces ou executando código JavaScript refletido no DOM.
