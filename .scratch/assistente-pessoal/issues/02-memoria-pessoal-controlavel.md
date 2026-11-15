# 02: Memória pessoal controlável

**What to build:** o assistente personaliza conversas futuras com memória privada do usuário, que ele pode consultar, corrigir, excluir ou desativar.

**Blocked by:** 01: Conversa privada autenticada.

**Status:** ready-for-agent

- [ ] O assistente salva automaticamente fatos duradouros e preferências úteis para conversas futuras.
- [ ] Inferências incertas e informações sensíveis não são salvas automaticamente.
- [ ] A memória de cada usuário permanece inacessível a outros usuários.
- [ ] O usuário pode consultar, corrigir e excluir itens de memória.
- [ ] Quando a memória está desativada, ela não é criada nem usada para responder.
- [ ] A memória é distinta do histórico e só o contexto relevante do usuário é usado na conversa.
- [ ] Os resultados são verificáveis pela API pública com usuários e provedor de IA simulados.
