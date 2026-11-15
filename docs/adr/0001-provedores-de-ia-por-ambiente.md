# Provedores de IA variam por ambiente

O desenvolvimento usa inferência local pela rede interna, enquanto a implantação usa um provedor em nuvem configurado para o ambiente, sem fallback silencioso. Essa escolha mantém o desenvolvimento local e simplifica a operação implantada; à nuvem só são enviados o contexto mínimo relevante e os documentos selecionados pelo usuário para a base de conhecimento. Antes de produção, o provedor deve garantir que esses dados não sejam usados para treinamento, além de explicitar retenção, região e exclusão.
