# 01: Conversa privada autenticada

**What to build:** uma pessoa autentica-se e conversa com o assistente, mantendo seu próprio histórico e usando o provedor de IA configurado para o ambiente.

**Blocked by:** None (can start immediately).

**Status:** ready-for-agent

- [ ] O usuário autentica-se por OIDC e pode criar, consultar e continuar suas próprias conversas.
- [ ] Um usuário não consegue consultar ou alterar conversas pertencentes a outro usuário.
- [ ] O histórico relevante da conversa é usado para responder, sem ser tratado como memória duradoura.
- [ ] O desenvolvimento usa Ollama na rede interna; a implantação usa o provedor em nuvem configurado, sem fallback silencioso.
- [ ] Antes do primeiro uso de IA em nuvem, o usuário recebe aviso claro sobre o processamento de conteúdo pelo provedor.
- [ ] Chamadas em nuvem incluem somente a solicitação e o contexto de conversa relevante; conteúdo sensível não aparece em logs ou traces.
- [ ] O provedor de produção só pode ser habilitado após verificação de não uso para treinamento, retenção, região e exclusão.
- [ ] O comportamento é verificável pela API pública com identidades e provedor de IA substituídos por dependências de teste.
