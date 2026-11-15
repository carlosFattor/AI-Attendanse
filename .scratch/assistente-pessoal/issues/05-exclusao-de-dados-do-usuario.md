# 05: Exclusão dos dados do usuário

**What to build:** o usuário remove seus dados pessoais do assistente, incluindo conversas, memória e conhecimento derivado, sem deixar esses dados disponíveis para consultas futuras.

**Blocked by:** 01: Conversa privada autenticada; 02: Memória pessoal controlável; 03: Ingestão de documentos privados.

**Status:** ready-for-agent

- [ ] O usuário consegue excluir conversas, memória e sua base de conhecimento.
- [ ] A exclusão de dados de uma conta remove os dados associados a essa identidade, sem afetar outros usuários.
- [ ] A exclusão da base remove o conhecimento derivado e impede seu uso em novas respostas.
- [ ] Cópias associadas no provedor de IA são excluídas quando o provedor oferecer suporte a essa operação.
- [ ] O usuário ou operador consegue distinguir uma solicitação de exclusão concluída de dados ainda sujeitos ao prazo documentado de expiração de backup.
- [ ] Logs, traces e respostas posteriores não expõem conteúdo excluído.
- [ ] O comportamento é verificável pela API pública com usuários e provedores simulados.
