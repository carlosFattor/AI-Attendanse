# Assistente pessoal multiusuário com conhecimento privado

**Status:** ready-for-agent

## Problem Statement

Usuários precisam de um assistente pessoal capaz de responder com base em conversas anteriores, memória e documentos de referência próprios. A solução precisa atender vários usuários sem misturar seus dados e funcionar com inferência local durante o desenvolvimento e com um provedor em nuvem na implantação. Hoje o repositório contém apenas documentação de arquitetura; não há aplicação implementada.

## Solution

Entregar um fluxo de conversa autenticado por OIDC, com histórico e memória privados por usuário, além de uma base de conhecimento reutilizável a partir dos documentos selecionados por ele. Um cliente local envia os documentos à aplicação principal, que processa a ingestão de forma assíncrona e informa o resultado. Os originais são temporários; o conhecimento derivado fica privado até ser excluído.

O provider de IA é configurado por ambiente: Ollama local acessível pela rede interna durante o desenvolvimento e um provedor em nuvem na implantação. A aplicação envia somente contexto relevante do usuário. Não há fallback silencioso entre providers.

## User Stories

1. As a user, I want to authenticate through OIDC, so that only my authenticated identity can access my assistant data.
2. As a user, I want each conversation to belong to my account, so that other users cannot read or alter it.
3. As a user, I want to continue a conversation with relevant prior messages available, so that I do not need to repeat context within that conversation.
4. As a user, I want conversation history to remain distinct from my long-term memory, so that past transcripts are not treated as durable personal facts.
5. As a user, I want the assistant to save durable facts and preferences automatically, so that future conversations can be personalized.
6. As a user, I want uncertain inferences and sensitive facts excluded from automatic memory, so that personalization does not create inappropriate or surprising records.
7. As a user, I want to view, correct, and delete individual memories, so that I can keep personalization accurate.
8. As a user, I want to disable memory, so that future conversations do not use or create long-term personal facts.
9. As a user, I want to select documents for my private knowledge base, so that the assistant can answer using information I choose to provide.
10. As a user, I want my selected documents sent from a local client to the main application, so that the application can process them without a separate worker service.
11. As a user, I want document ingestion to continue asynchronously after upload, so that large documents do not require one long-lived request.
12. As a user, I want to see whether ingestion is pending, successful, or failed, so that I know when selected documents are ready for questions.
13. As a user, I want failed ingestion to be retried only a limited number of times, so that temporary copies of my originals are not retained indefinitely.
14. As a user, I want original documents discarded after successful processing or final failure, so that the uploaded files are not kept unnecessarily.
15. As a user, I want knowledge derived from my selected documents to remain available across conversations, so that I can reuse my private knowledge base.
16. As a user, I want my knowledge base and its derived content to be inaccessible to other users, so that my documents remain private.
17. As a user, I want the assistant to retrieve only relevant material from my knowledge base, so that answers are grounded without sending the entire collection for every question.
18. As a user, I want the assistant to say when my knowledge base does not contain supporting information, so that it does not present unsupported answers as document-backed facts.
19. As a user, I want the development environment to use the locally hosted model provider, so that development inference stays on the internal network.
20. As an operator, I want the deployed environment to use its configured cloud provider, so that provider selection is explicit and appropriate to that environment.
21. As an operator, I want provider failures to be reported without silently switching environments or providers, so that data is not unexpectedly sent to another service.
22. As a user, I want a clear notice before my first cloud-backed use, so that I understand that relevant prompts, memory, history, and retrieved document passages may be processed by an external provider.
23. As a user, I want only the minimum relevant context for my request sent to the cloud provider, so that unrelated conversation and knowledge-base content are not disclosed.
24. As an operator, I want the cloud provider not to use submitted content for training and to have explicit retention, region, and deletion terms, so that user data handling can be evaluated before production.
25. As a user, I want my knowledge-base documents sent for cloud embeddings only when I select them for ingestion, so that unselected documents are not submitted to the provider.
26. As a user, I want to delete my conversations, memory, and knowledge base, so that I can remove my personal data from the application.
27. As a user, I want deletion to include derived knowledge and provider copies where supported, so that deleting source data also removes its usable derivatives.
28. As an operator, I want backup expiration and provider deletion behavior documented, so that deletion requests have a clear completion boundary.
29. As a user, I want identifiers or credentials authorized for the ingestion route to be checked in addition to my authenticated identity, so that route access can be restricted without letting a client choose another user's data owner.
30. As a user, I want sensitive conversation and document contents excluded from operational logs and traces, so that observability does not expose private information.

## Implementation Decisions

- The application supports multiple users. Conversation, memory, and knowledge-base ownership is private to the authenticated user.
- OIDC authenticates the user. The main application derives data ownership from that identity; an ID authorized for the ingestion route is an additional authorization check, not a substitute for user identity.
- Conversation history and long-term memory are separate concepts. Memory is saved automatically only for durable facts and preferences; uncertain inferences and sensitive facts are excluded. Users can inspect, correct, delete, or disable memory.
- A user's knowledge base is reusable across that user's conversations. It contains only documents selected by that user.
- A local client submits selected documents to the main application. The main application processes ingestion asynchronously; no separate worker service is required. The user can observe ingestion status and outcome.
- Original documents are retained only temporarily for a bounded number of retries and discarded after successful processing or final failure. Derived knowledge remains private to its owner until deletion.
- In the development environment, model requests use local Ollama reachable over the internal network. In the deployed environment, generation and embeddings use the configured cloud provider when it supports both capabilities. Provider choice is environment configuration, not silent runtime fallback.
- Candidate model names and quantizations in the architecture document are provisional and must be benchmarked on the target hardware; this spec does not lock a production model or cloud vendor.
- For cloud-backed requests, send only the current request and relevant conversation history, memory, and retrieved passages. Documents selected for ingestion are sent to the cloud embedding provider in the deployed environment.
- Before production, verify the provider's no-training terms, data-retention controls, processing region, and deletion behavior. Present a clear notice before first cloud-backed use.
- User deletion removes conversation history, memory, knowledge-base derivatives, and provider copies where supported. Document backup expiration and the limits of provider-side deletion.
- Do not record sensitive prompts, document contents, conversation contents, or credentials in operational logs and traces.
- Preserve application-owned AI ports such as `ChatModelPort` and `EmbeddingModelPort` so use cases do not depend on a vendor-specific provider API.

## Testing Decisions

- Use one primary seam: exercise behavior through the public Quarkus API. Authenticate as multiple test users and replace external identity and AI providers with test doubles at their existing architectural boundaries.
- Test externally observable behavior rather than internal classes, implementation details, or the chosen persistence mechanism.
- Verify that one user cannot access another user's conversations, memory, ingestion jobs, or knowledge base, including when a client supplies a mismatched route identifier.
- Verify conversation history and long-term memory are separate, automatic memory follows the agreed inclusion rules, and user controls for viewing, correcting, deleting, and disabling memory work.
- Verify selected-document ingestion reports pending, success, and final failure; original uploads are not retained after success or exhausted retries; derived knowledge supports later questions and is removed by deletion.
- Verify answers use only the authenticated user's selected knowledge, unsupported questions are not represented as document-backed, and cloud-provider test doubles receive only the minimum relevant context and selected ingestion content.
- Verify environment configuration selects local Ollama for development and the configured cloud provider for deployment, with no silent fallback to another provider.
- Verify deletion removes the user's application data and invokes provider deletion where supported. Verify sensitive content is absent from operational logs and traces.
- No application code or prior test suite exists in the repository. Establish API-level integration tests alongside the application; there is no existing test prior art to reuse.

## Out of Scope

- Selecting or contracting with a specific cloud vendor, and verifying its live contractual terms.
- Finalizing model versions, quantization, embedding dimensions, GPU sizing, and production concurrency before hardware benchmarks.
- Sharing conversations, memory, or knowledge bases between users or creating organization-level shared collections.
- Public exposure of the local Ollama runtime or automatic failover from local inference to cloud.
- Keeping original uploaded documents as a permanent archive.
- Building an independent worker service for document ingestion.

## Further Notes

- The repository currently contains architecture and agent documentation, but no application source, API, schema, or test suite. Implementation choices must be reconciled with the actual application scaffold when it is added.
- The architecture document's Qwen3 and Ollama recommendations remain provisional. Provider support for both generation and embeddings, along with privacy and deletion guarantees, must be confirmed before production.
- Backup retention cannot be assumed to provide immediate physical erasure; document its expiration behavior to users and operators.
