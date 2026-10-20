# Especificação de Testes: US01 - Recuperação de Senha por E-mail

---

### Cenário: [CN-01] - Solicitação e Recuperação de Senha com E-mail Cadastrado

- **Categoria:** Caminho Feliz
- **Objetivo:** Validar o fluxo completo de navegação a partir da tela de login, solicitação de recuperação de senha para um usuário existente e utilização do link com token válido dentro do prazo de 15 minutos.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-001] - Navegar para a tela de recuperação através do link "Esqueci minha senha"

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Usuário não autenticado na tela de login da plataforma.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está na tela de login da plataforma
  - **Quando** visualiza as opções de acesso e clica no link "Esqueci minha senha"
  - **Então** o sistema redireciona o usuário para a tela/formulário de recuperação de senha contendo o campo de e-mail e o botão de envio
- **Resultado Esperado Detalhado:** O link "Esqueci minha senha" está visível e acessível na tela de login, redirecionando corretamente para a rota de recuperação de senha.

##### Caso de Teste: [CT-002] - Solicitar recuperação de senha com e-mail válido e cadastrado na base

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:**
  - Usuário possui conta ativa com o e-mail `usuario.valido@dominio.com` cadastrado na base de dados.
  - O IP do usuário possui menos de 5 tentativas recentes no formulário.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está na tela de recuperação de senha
  - **Quando** preenche o campo de e-mail com `"usuario.valido@dominio.com"`
  - **E** submete o formulário de recuperação
  - **Então** o sistema exibe a mensagem `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."`
  - **E** dispara um e-mail contendo um link de redefinição com um token único válido por 15 minutos
- **Resultado Esperado Detalhado:** A interface exibe a mensagem padrão sem expor dados sensíveis, a API retorna status HTTP 200/202 e o serviço de mensageria envia o e-mail com token único vinculado ao usuário e timestamp de expiração de 15 minutos.

##### Caso de Teste: [CT-003] - Acessar link de recuperação com token válido dentro do prazo de 15 minutos

- **Tipo:** Funcional
- **Prioridade:** Alta
- **Pré-condições:** Solicitação de recuperação realizada há menos de 15 minutos para `usuario.valido@dominio.com`, com token único ainda não utilizado.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário recebeu o e-mail de recuperação de senha há 5 minutos
  - **Quando** acessa o link contendo o token único antes de completar 15 minutos da emissão
  - **Então** o sistema valida o token com sucesso e exibe o formulário para definição da nova senha
- **Resultado Esperado Detalhado:** O token é reconhecido como ativo e íntegro pelo backend, permitindo ao usuário prosseguir com a criação da nova credencial.

---

### Cenário: [CN-02] - Validação de Formato e Obrigatoriedade do Campo de E-mail

- **Categoria:** Caminho Alternativo
- **Objetivo:** Garantir que o formulário de recuperação valide entradas vazias, formatos de e-mail inválidos e normalização de caracteres antes de processar a requisição.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-004] - Tentar solicitar recuperação com o campo de e-mail vazio

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Usuário na tela de recuperação de senha.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está no formulário de recuperação de senha
  - **Quando** mantém o campo de e-mail em branco
  - **E** aciona o botão de enviar
  - **Então** o sistema impede a submissão do formulário e exibe uma mensagem de validação informando que o campo de e-mail é obrigatório
- **Resultado Esperado Detalhado:** O formulário destaca o campo obrigatório e não dispara envio de e-mail no backend.

##### Caso de Teste: [CT-005] - Tentar solicitar recuperação informando e-mail com formato inválido

- **Tipo:** Validação
- **Prioridade:** Média
- **Pré-condições:** Usuário na tela de recuperação de senha.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está no formulário de recuperação de senha
  - **Quando** informa um endereço de e-mail com sintaxe inválida (ex.: `"usuario_sem_arroba.com"`, `"usuario@"`, `"@dominio.com"` ou `"usuario@dominio"`)
  - **E** tenta submeter o formulário
  - **Então** o sistema rejeita a entrada e exibe uma mensagem de erro indicando que o usuário deve informar um e-mail válido
- **Resultado Esperado Detalhado:** A validação client-side e server-side bloqueia formatos fora do padrão de e-mail (RFC 5322), retornando HTTP 400 caso a requisição chegue à API.

##### Caso de Teste: [CT-006] - Solicitar recuperação informando e-mail cadastrado com letras maiúsculas e espaços nas extremidades

- **Tipo:** Validação
- **Prioridade:** Baixa
- **Pré-condições:** E-mail `usuario.valido@dominio.com` cadastrado em letras minúsculas na base.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está no formulário de recuperação de senha
  - **Quando** preenche o campo de e-mail com `"  USUARIO.VALIDO@DOMINIO.COM  "`
  - **E** submete o formulário
  - **Então** o sistema aplica normalização (`trim` e `lowercase`), localiza o cadastro e envia o link de recuperação normalmente
- **Resultado Esperado Detalhado:** O sistema trata espaços acidentais e diferenças de caixa alta/baixa como equivalentes ao e-mail cadastrado, exibindo a mensagem padrão e enviando o token.

---

### Cenário: [CN-03] - Prevenção de Enumeração de Usuários (E-mail Não Cadastrado)

- **Categoria:** Exceção e Segurança
- **Objetivo:** Verificar que o sistema protege a privacidade dos usuários cadastrados ao não revelar se um endereço de e-mail existe ou não na plataforma.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-007] - Solicitar recuperação para um e-mail válido não existente na base

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** O e-mail `inexistente@dominio.com` possui sintaxe válida, mas não está cadastrado na base de dados.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário está na tela de recuperação de senha
  - **Quando** informa o e-mail `"inexistente@dominio.com"`
  - **E** submete o formulário
  - **Então** o sistema exibe exatamente a mensagem genérica `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."`
  - **E** nenhum e-mail ou token é gerado internamente
- **Resultado Esperado Detalhado:** A resposta visual, o código de status HTTP da API e a estrutura do payload de retorno devem ser idênticos aos de um e-mail existente, impedindo ataques de enumeração de contas (User Enumeration).

---

### Cenário: [CN-04] - Regras de Segurança, Expiração e Unicidade do Token

- **Categoria:** Exceção e Segurança
- **Objetivo:** Garantir que o token de recuperação expire rigorosamente após 15 minutos, seja de uso único e não permita adulteração ou reutilização.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-008] - Tentar utilizar link de recuperação após o limite de 15 minutos (Token Expirado)

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Token de recuperação gerado há mais de 15 minutos (ex.: 15 minutos e 1 segundo).
- **Passos de Execução (BDD):**
  - **Dado** que o usuário solicitou a recuperação de senha e recebeu um link com token único
  - **Quando** aguarda mais de 15 minutos após a geração do token
  - **E** acessa o link de recuperação recebido por e-mail
  - **Então** o sistema bloqueia a redefinição de senha e informa que o link/token expirou, orientando o usuário a realizar uma nova solicitação
- **Resultado Esperado Detalhado:** O backend rejeita o token expirado (HTTP 401/403 ou 410), impedindo qualquer alteração de senha através dele.

##### Caso de Teste: [CT-009] - Tentar reutilizar um token único que já foi consumido

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Usuário já redefiniu sua senha utilizando o token recebido há menos de 15 minutos.
- **Passos de Execução (BDD):**
  - **Dado** que o token de recuperação já foi utilizado uma vez com sucesso para redefinir a senha
  - **Quando** o usuário (ou um atacante) acessa novamente o mesmo link dentro da janela de 15 minutos
  - **Então** o sistema recusa o acesso e exibe mensagem de token inválido ou já utilizado
- **Resultado Esperado Detalhado:** O token é invalidado imediatamente após o primeiro uso bem-sucedido, garantindo a propriedade de token de uso único (Single-Use Token).

##### Caso de Teste: [CT-010] - Invalidar token anterior ao solicitar um novo link de recuperação

- **Tipo:** Segurança
- **Prioridade:** Média
- **Pré-condições:** Usuário solicitou recuperação de senha (`Token A`) e, 2 minutos depois, solicitou novamente (`Token B`).
- **Passos de Execução (BDD):**
  - **Dado** que o usuário possui duas solicitações consecutivas de recuperação para o mesmo e-mail (`Token A` seguido de `Token B`)
  - **Quando** tenta utilizar o link referente ao `Token A` (primeira solicitação)
  - **Então** o sistema rejeita o `Token A` como inválido e mantém apenas o `Token B` ativo por 15 minutos
- **Resultado Esperado Detalhado:** Apenas o último token emitido para a conta permanece válido, revogando tokens pendentes anteriores.

---

### Cenário: [CN-05] - Limite de Tentativas e Proteção contra Abuso (Rate Limit por IP)

- **Categoria:** Exceção e Segurança
- **Objetivo:** Assegurar que o formulário bloqueie novas submissões após 5 tentativas consecutivas originadas do mesmo endereço IP.

**Caso de Teste (CT):**

##### Caso de Teste: [CT-011] - Permitir até a 5ª tentativa consecutiva de recuperação pelo mesmo IP

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** Usuário (ou script) originando requisições do mesmo endereço IP (`192.0.2.10`), iniciando com contador de tentativas zerado.
- **Passos de Execução (BDD):**
  - **Dado** que o endereço IP `192.0.2.10` realizou 4 tentativas consecutivas no formulário de recuperação
  - **Quando** submete a 5ª tentativa consecutiva com um e-mail válido
  - **Então** o sistema processa a 5ª solicitação normalmente e exibe `"Se o e-mail estiver cadastrado, você receberá as instruções em breve."`
- **Resultado Esperado Detalhado:** O limite exato de 5 tentativas é respeitado antes da ativação do bloqueio (teste de valor limite no limiar superior permitido).

##### Caso de Teste: [CT-012] - Bloquear o formulário após 5 tentativas consecutivas com o mesmo IP (6ª tentativa)

- **Tipo:** Segurança
- **Prioridade:** Alta
- **Pré-condições:** O endereço IP `192.0.2.10` já efetuou 5 tentativas consecutivas de recuperação de senha.
- **Passos de Execução (BDD):**
  - **Dado** que o usuário atingiu 5 tentativas consecutivas no formulário de recuperação a partir do mesmo IP
  - **Quando** tenta submeter uma 6ª solicitação (com e-mail cadastrado ou não)
  - **Então** o sistema bloqueia o envio do formulário e exibe mensagem informando que o limite de tentativas foi excedido
  - **E** a API rejeita a requisição com status HTTP `429 Too Many Requests` sem processar ou disparar e-mails
- **Resultado Esperado Detalhado:** O formulário fica bloqueado para aquele IP após a 5ª tentativa, mitigando ataques de força bruta, flooding de caixa de entrada e enumeração massiva.

---
