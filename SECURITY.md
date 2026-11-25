# Segurança

Este projeto foi construído e avaliado seguindo uma matriz própria de segurança de plataformas, baseada em **OWASP Top 10:2025**, **OWASP API Security Top 10:2023**, **OWASP Top 10 para aplicações com LLM (2026)** e na **LGPD**. A avaliação detalhada, com evidências e plano de ação, é mantida fora do repositório.

## Controles implementados

### Dados e autorização
- **Row Level Security** em todas as tabelas, com políticas por operação e triggers que forçam o dono de cada linha.
- **Sem chave de administrador** no projeto: cada consulta roda com a identidade do próprio usuário, então o banco decide o que cada pessoa pode ler e gravar.
- Permissão de escrita por coluna: a pessoa altera só os próprios horários de lembrete e o próprio plano de estudo (total validado no banco), nunca a cota de IA.
- Tarefas automáticas (lembrete diário) rodam com um papel restrito, sem login e sem acesso direto às tabelas, por funções com o mínimo necessário.
- Foto de perfil em bucket privado com RLS por pasta: cada pessoa acessa só o próprio arquivo; tamanho (5 MB) e tipos de imagem limitados no servidor.
- Isolamento entre contas verificado com testes no banco.

### Sessão e login
- Refresh token em cookie com `HttpOnly`, `Secure` e `SameSite=Strict`; access token de curta duração apenas em memória.
- "Lembrar de mim" altera só a duração do cookie; a senha nunca é armazenada.
- Proteção contra CSRF (SameSite, verificação de origem e cabeçalho obrigatório).
- Mensagens de login e de recuperação de senha que não revelam se uma conta existe.
- Logout revoga a sessão no servidor.
- Troca de senha dentro do app exige a senha atual; política de senha forte também no servidor.

### API e IA
- Validação e limite de tamanho de entradas; erros genéricos para o cliente; logs sem conteúdo do usuário.
- Cotas de uso da IA por pessoa e CORS restrito ao domínio.
- Texto do usuário tratado como dado no prompt; saída da IA em JSON validado por schema (structured outputs), depois validada de novo, truncada e escapada antes de ser exibida.
- Autoteste da IA protegido por segredo; usa só dados de exemplo e não acessa dados de ninguém.
- Chave da API de IA disponível apenas no servidor.
- Notificações: chaves e segredos guardados no cofre do Supabase; endereços de push restritos aos serviços oficiais (contra SSRF); limite de aparelhos por pessoa.

### Navegador e aplicativo
- Content Security Policy estrita, HSTS, proteção contra iframe, `nosniff`, `Referrer-Policy` e `Permissions-Policy`.
- Build sem source maps; service worker nunca armazena respostas de API; a cópia offline e a fila de envio não contêm credenciais e são apagadas ao sair.

### Processo
- Varredura de segredos (gitleaks) e testes automatizados (API e e2e) a cada push.
- Nenhuma credencial privada no repositório; variáveis de ambiente apenas nos provedores.

## Como reportar uma vulnerabilidade

Use o recurso **"Report a vulnerability"** na aba *Security* deste repositório (relato privado). Por favor, não abra issue pública com detalhes de uma falha.
