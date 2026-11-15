# Arquitetura de Inferência de IA Local

**Projeto:** Personal AI Assistant\
**Versão:** 1.0\
**Tipo:** Documento técnico de referência

## Objetivo

Definir uma arquitetura inicial para executar modelos de linguagem (LLM)
e modelos de embeddings em um servidor separado, acessível pela rede
local a partir da aplicação Quarkus.

## 1. Premissas e hardware conhecido

  -----------------------------------------------------------------------
  Item                    Informação              Impacto
  ----------------------- ----------------------- -----------------------
  RAM                     32 GB                   Permite manter runtime,
                                                  sistema e processamento
                                                  auxiliar; não elimina o
                                                  limite de VRAM.

  GPU                     NVIDIA dedicada         Possibilita aceleração
                                                  por GPU; é necessário
                                                  confirmar modelo,
                                                  arquitetura e driver.

  VRAM                    12 GB                   Adequada para começar
                                                  com LLMs quantizados na
                                                  faixa de 8B, dependendo
                                                  do contexto.

  Localização             Outro computador        Inferência desacoplada
                                                  do backend e
                                                  comunicação pela rede
                                                  local.

  CPU / sistema           Não informados          Necessários para
  operacional / rede                              dimensionar offload,
                                                  instalação e latência.
  -----------------------------------------------------------------------

**Premissa de segurança:** o servidor de inferência deve ser acessível
somente por hosts autorizados da rede local. Não publicar a porta do
runtime diretamente na Internet.

## 2. Recomendação executiva

Adotar inicialmente:

-   **Runtime:** Ollama.
-   **LLM de geração:** Qwen3 8B quantizado em Q4_K_M.
-   **Embeddings:** Qwen3-Embedding-0.6B, dimensão 1024.
-   **Comunicação:** HTTP pela LAN.
-   **Integração:** LangChain4j no Quarkus, por meio de adaptadores
    próprios.
-   **Concorrência inicial:** uma geração simultânea, até concluir os
    testes de capacidade.

A escolha é provisória até executar benchmark no hardware real. A VRAM
de 12 GB pode comportar modelos quantizados próximos de 8B, mas
contexto, cache KV, runtime e concorrência também consomem memória.

## 3. Comparação dos modelos de geração

As estimativas de memória são indicativas, não garantias. O consumo real
depende da quantização, contexto, cache KV, batch, runtime e
configuração de GPU.

  -------------------------------------------------------------------------
  Modelo                  Configuração para       Avaliação para 12 GB
                          testar                  
  ----------------------- ----------------------- -------------------------
  Qwen3 8B                Q4_K_M; comparar Q5_K_M Primeira escolha. Boa
                                                  margem relativa para
                                                  contexto e cache;
                                                  adequado para português,
                                                  síntese e RAG.

  Gemma 3 12B             Q4_K_M                  Alternativa de benchmark.
                                                  Pode ficar próximo do
                                                  limite e reduzir margem
                                                  para contexto e
                                                  concorrência.

  Qwen3 14B               Q4                      Experimental/limítrofe.
                                                  Pode exigir offload e
                                                  limitar contexto ou
                                                  desempenho.

  Qwen3 32B               Q4                      Não recomendado para esta
                                                  GPU como configuração
                                                  inicial; memória exigida
                                                  muito acima de 12 GB.
  -------------------------------------------------------------------------

### Recomendação

Começar com **Qwen3 8B Q4_K_M**. Comparar com Q5_K_M se houver VRAM
suficiente. Testar Gemma 3 12B Q4_K_M como candidato alternativo,
monitorando pico de VRAM e latência.

Não selecionar o modelo somente pelo número de parâmetros. A decisão
deve considerar qualidade das respostas, fidelidade ao contexto RAG,
tempo até o primeiro token, tokens por segundo e consumo de memória.

## 4. Modelos de embeddings

O modelo de embeddings transforma documentos e perguntas em vetores
usados na recuperação semântica do RAG.

  -------------------------------------------------------------------------
  Modelo                                     Dimensão Observações
  ---------------------- ---------------------------- ---------------------
  Qwen3-Embedding-0.6B                       Até 1024 Candidato principal;
                                                      multilíngue, suporta
                                                      instruções e
                                                      dimensões
                                                      configuráveis.

  BGE-M3                                         1024 Alternativa robusta;
                                                      oferece
                                                      possibilidades de
                                                      recuperação densa e
                                                      híbrida.

  EmbeddingGemma                                  768 Opção compacta para
                                                      benchmark de custo e
                                                      desempenho.
  -------------------------------------------------------------------------

### Recomendação

Começar com **Qwen3-Embedding-0.6B com dimensão 1024**.

O modelo usado para indexar documentos deve ser compatível com o modelo
usado para vetorizar consultas. Uma troca posterior de modelo ou
configuração de embeddings exige reindexar os chunks.

A execução dos embeddings na CPU é uma hipótese inicial para reservar
VRAM ao LLM, não uma regra. Validar throughput e latência. Se a CPU for
lenta, testar GPU com controle de concorrência.

## 5. Arquitetura lógica proposta

O serviço de inferência fica em um computador separado. O Quarkus mantém
a orquestração da conversa, autenticação, memória, RAG e regras de
negócio. O runtime remoto executa os modelos.

``` text
Visitante
   |
   v
React / TypeScript
   |
   v
Quarkus API (Servidor A)
   |-- autenticação OIDC
   |-- histórico e memória
   |-- recuperação RAG / PostgreSQL + pgvector
   |-- montagem de prompt
   |-- ChatModelPort / EmbeddingModelPort
   |
   | HTTP pela LAN
   v
Servidor B: Inferência
   |
   +-- Ollama API
        |-- Qwen3 8B (geração)
        |-- Qwen3-Embedding-0.6B (vetorização)
        |
        +-- GPU NVIDIA 12 GB / RAM 32 GB
```

### Responsabilidades

-   **Quarkus:** autenticação, autorização, gestão de conversas,
    memória, RAG, prompt, políticas, limites por usuário,
    observabilidade e tratamento de erros.
-   **Servidor de inferência:** carregamento dos modelos, execução de
    geração e embeddings, controle de recursos e exposição da API
    interna.
-   **PostgreSQL + pgvector:** persistência de chunks, vetores,
    metadados, histórico e memória conforme o modelo de dados da
    aplicação.
-   **Observabilidade:** instrumentar chamadas e medir latência, tokens,
    erros e consumo de recursos sem registrar conteúdo sensível.

## 6. Runtime e comunicação

Ollama é a escolha inicial pela simplicidade operacional, API HTTP e
facilidade de troca de modelos. O backend deve depender de interfaces
próprias, não de detalhes específicos do Ollama.

### Configuração de rede

1.  Configurar o runtime para escutar no endereço de rede apropriado, e
    não apenas em loopback, quando o acesso remoto for necessário.
2.  Restringir a porta TCP `11434` no firewall para o IP do servidor
    Quarkus ou para uma VLAN/rede confiável.
3.  Não fazer port forwarding no roteador nem expor a API diretamente à
    Internet.
4.  Se houver necessidade de atravessar redes não confiáveis, usar VPN
    ou proxy reverso com autenticação e TLS.
5.  Definir DNS/IP reservado para o servidor de inferência e timeout
    apropriado no cliente.

### Exemplo de configuração do Quarkus

``` properties
ai.provider=ollama

ai.ollama.base-url=http://192.168.1.100:11434

ai.ollama.chat.model=qwen3:8b
ai.ollama.embedding.model=qwen3-embedding:0.6b

ai.chat.timeout=120s
ai.embedding.timeout=30s
```

Os nomes de modelo são exemplos. Antes de fixá-los em configuração,
verificar as tags disponíveis no registry/runtime e registrar a versão
exata utilizada nos ambientes.

## 7. Integração com Onion / Hexagonal Architecture

Estrutura sugerida:

``` text
application/
  port/
    ChatModelPort.java
    EmbeddingModelPort.java
    ModelHealthPort.java
  service/
    GenerateAnswerService.java
    CreateEmbeddingService.java

infrastructure/
  ai/
    ollama/
      OllamaChatAdapter.java
      OllamaEmbeddingAdapter.java
      OllamaHealthAdapter.java
      OllamaConfiguration.java
```

As interfaces devem expressar capacidades da aplicação. O adaptador
Ollama traduz chamadas para a API do runtime. Assim, trocar Ollama por
outro provider não exige alterar os casos de uso.

### Responsabilidades das portas

-   `ChatModelPort`: receber mensagens, parâmetros de geração e retornar
    resposta/uso de tokens.
-   `EmbeddingModelPort`: transformar texto em vetor e informar
    modelo/dimensão utilizados.
-   `ModelHealthPort`: verificar disponibilidade e capacidade de
    inferência do provider.

## 8. Concorrência, resiliência e operação

-   **Concorrência inicial:** limitar a uma geração simultânea até medir
    VRAM, throughput e latência. Ajustar gradualmente com testes.
-   **Fila limitada:** aceitar quantidade controlada de solicitações
    pendentes; rejeitar ou responder com indisponibilidade temporária
    quando saturada.
-   **Timeouts:** configurar separadamente geração e embeddings;
    considerar o tempo de carregamento inicial do modelo.
-   **Circuit breaker:** evitar cascata de falhas quando o servidor de
    inferência estiver indisponível.
-   **Retry:** usar apenas em falhas transitórias e com política
    limitada; evitar repetir geração não idempotente
    indiscriminadamente.
-   **Warm-up:** avaliar manter o modelo carregado para reduzir latência
    inicial, observando o custo de memória.
-   **Backpressure:** impedir que o backend acumule requisições
    ilimitadas enquanto a GPU está ocupada.
-   **Health check:** distinguir serviço HTTP disponível de modelo
    efetivamente carregado e apto a inferir.

## 9. Plano de benchmark

Executar os testes no servidor definitivo, com versões de runtime,
modelo, quantização e parâmetros registrados. Usar perguntas e
documentos sintéticos ou autorizados, evitando dados pessoais reais
durante a calibração.

### Critérios e pesos iniciais

  ------------------------------------------------------------------------
  Critério                              Peso inicial Medição
  --------------------- ---------------------------- ---------------------
  Qualidade em                                   25% Avaliação humana e
  português                                          conjunto de perguntas
                                                     padronizadas.

  Fidelidade ao RAG                              25% Resposta sustentada
                                                     pelos chunks;
                                                     reconhecer ausência
                                                     de evidência.

  Síntese e raciocínio                           20% Perguntas que
                                                     combinam informações
                                                     de diferentes
                                                     trechos.

  Time to first token                            10% Tempo até o primeiro
                                                     token da resposta.

  Tokens por segundo                             10% Velocidade de geração
                                                     após início.

  Memória                                        10% Pico de VRAM/RAM,
                                                     incluindo contexto e
                                                     concorrência.
  ------------------------------------------------------------------------

### Casos mínimos de teste

1.  Pergunta respondível por um único documento.
2.  Pergunta que exige combinar dois ou mais documentos.
3.  Pergunta sem resposta na base: o modelo deve admitir que não
    encontrou evidência.
4.  Pergunta dependente do histórico da conversa.
5.  Prompt longo próximo do limite de contexto definido.
6.  Duas solicitações simultâneas e teste progressivo de concorrência.
7.  Teste de indisponibilidade, reinício e descarregamento do modelo.

Registrar em cada execução:

-   Modelo/tag e quantização.
-   Tamanho do contexto e parâmetros de geração.
-   Versão do Ollama, driver NVIDIA e CUDA.
-   Temperatura e concorrência.
-   Tokens por segundo, TTFT e latência total.
-   Pico de VRAM/RAM.
-   Avaliação da resposta.

## 10. Observabilidade e segurança

### Observabilidade

Criar spans para:

-   Chamada LLM.
-   Geração de embeddings.
-   Recuperação RAG.
-   Montagem do prompt.
-   Persistência.

Métricas recomendadas:

-   Latência p50/p95.
-   Tokens de entrada e saída.
-   Tokens por segundo.
-   Erros por provider e timeouts.
-   Tamanho da fila e concorrência.
-   Consumo de GPU, quando disponível.

### Segurança

-   Evitar IDs de usuário/conversa/documento como labels de alta
    cardinalidade.
-   Não enviar conteúdo integral de prompts, documentos, conversas,
    tokens de acesso ou credenciais para logs e traces.
-   Separar telemetria operacional de avaliação de qualidade, com
    política explícita de retenção e acesso.
-   Proteger a API de inferência por firewall; não confiar apenas no
    fato de estar em uma LAN.
-   Manter o servidor de inferência sem exposição pública direta.

## 11. Decisões e pendências

  -----------------------------------------------------------------------
  Decisão                 Estado                  Próxima ação
  ----------------------- ----------------------- -----------------------
  Runtime Ollama          Proposta                Instalar no servidor e
                                                  validar acesso remoto
                                                  restrito.

  LLM Qwen3 8B Q4         Baseline provisória     Comparar Q4_K_M e
                                                  Q5_K_M; benchmark
                                                  contra Gemma 3 12B.

  Embedding Qwen3 0.6B    Baseline provisória     Medir CPU vs GPU e
                                                  validar qualidade em
                                                  português.

  Dimensão vetorial 1024  Provisória              Confirmar dimensão
                                                  efetiva do modelo e
                                                  fixar no schema.

  CPU e GPU exatas        Pendente                Obter modelos exatos
                                                  para dimensionamento.

  Sistema operacional e   Pendente                Confirmar Linux/Windows
  rede                                            e Ethernet/Wi-Fi.

  Concorrência            Uma geração inicial     Aumentar apenas após
                                                  testes de saturação.
  -----------------------------------------------------------------------

## 12. Sequência de implementação

1.  Identificar CPU, GPU, sistema operacional, driver NVIDIA e
    velocidade da rede.
2.  Instalar Ollama e validar inferência local no servidor B.
3.  Configurar acesso HTTP restrito pela LAN e testar do servidor A.
4.  Implementar adaptadores `ChatModelPort` e `EmbeddingModelPort` no
    Quarkus.
5.  Integrar RAG com pgvector e verificar compatibilidade de embeddings.
6.  Executar benchmark, ajustar contexto, quantização e concorrência.
7.  Instrumentar OpenTelemetry e criar dashboards/alertas.

## 13. Referências técnicas

-   Ollama --- documentação de API:
    <https://github.com/ollama/ollama/blob/main/docs/api.md>
-   Ollama --- FAQ e configuração de rede: <https://docs.ollama.com/faq>
-   Qwen3 --- modelos e model cards:
    <https://huggingface.co/collections/Qwen/qwen3-67dd247413f0e2e4f653967f>
-   Qwen3 Embedding --- documentação/model cards:
    <https://huggingface.co/collections/Qwen/qwen3-embedding-67dd247413f0e2e4f653967f>
-   BGE-M3 --- model card: <https://huggingface.co/BAAI/bge-m3>
-   Google Gemma --- modelos: <https://ai.google.dev/gemma>

------------------------------------------------------------------------

**Nota:** especificações de memória, tags, compatibilidade e desempenho
mudam conforme versões de modelos e runtimes. Validar a documentação e
executar medições no hardware real antes de assumir capacidade de
produção.
