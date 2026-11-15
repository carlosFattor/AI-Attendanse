# 03: Ingestão de documentos privados

**What to build:** o usuário envia documentos selecionados por um cliente local e acompanha o processamento até que o conhecimento derivado esteja disponível em sua base privada.

**Blocked by:** 01: Conversa privada autenticada.

**Status:** ready-for-agent

- [ ] O cliente local envia à aplicação principal somente documentos selecionados pelo usuário autenticado.
- [ ] A identidade OIDC define o proprietário; um ID autorizado para a rota de ingestão é uma verificação adicional e não pode escolher outro proprietário.
- [ ] A aplicação principal aceita o envio e processa a ingestão de forma assíncrona, sem exigir um worker separado.
- [ ] O usuário consegue consultar estados de processamento pendente, sucesso e falha definitiva.
- [ ] Documentos selecionados são extraídos e preparados para busca semântica, usando o provedor de embeddings configurado para o ambiente.
- [ ] O documento original fica disponível apenas temporariamente para um número limitado de tentativas e é descartado após sucesso ou falha definitiva.
- [ ] O conhecimento derivado pertence somente ao usuário autenticado e pode ser reutilizado em conversas futuras.
- [ ] Conteúdo de documentos não é exposto em logs ou traces, e documentos não selecionados não são enviados ao provedor.
- [ ] Os resultados são verificáveis pela API pública com usuários e provedor de embeddings simulados.
