# 04: Respostas com base no conhecimento

**What to build:** o usuário recebe respostas fundamentadas apenas no conteúdo relevante de sua própria base de conhecimento e é informado quando ela não contém evidência suficiente.

**Blocked by:** 01: Conversa privada autenticada; 03: Ingestão de documentos privados.

**Status:** ready-for-agent

- [ ] Uma pergunta recupera somente conteúdo da base de conhecimento do usuário autenticado.
- [ ] A resposta usa trechos relevantes e não envia a base completa ao provedor.
- [ ] Conteúdo de uma base de conhecimento não pode aparecer nas respostas de outro usuário.
- [ ] Quando não há evidência suficiente, o assistente informa essa limitação em vez de apresentar a resposta como fundamentada nos documentos.
- [ ] O conteúdo enviado ao provedor contém apenas a pergunta e os trechos relevantes necessários para respondê-la.
- [ ] A seleção e o resultado da recuperação são verificáveis por comportamento observável na API, usando um provedor simulado.
