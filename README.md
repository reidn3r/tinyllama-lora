# LoRA em LLMs para adaptação de estilo linguístico

> 📄 Paper LoRA: [link](https://arxiv.org/abs/2106.09685)

> 📂 Dataset: [link](https://huggingface.co/datasets/shiv213/Automatic-Sarcasm-Detection-Twitter)

> 🤗 Modelo: [link](https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0)

> ☁️ Desenvolvido no [Kaggle](https://www.kaggle.com) com GPU Tesla T4


O projeto aplica fine-tuning supervisionado (SFT) no modelo **TinyLlama-1.1B-Chat-v1.0** para a tarefa de geração de respostas sarcásticas a partir de conversas contextualizadas. O dataset utilizado contém 2.500 exemplos de treino, todos da classe `SARCASM`, com pré-processamento para remoção de menções, URLs e hashtags. O modelo base é carregado com quantização **4-bit NF4** via `BitsAndBytesConfig`, reduzindo o consumo de memória sem sacrifício relevante de qualidade, viabilizando o treinamento dentro dos limites da GPU disponível no Kaggle.

O modelo foi escolhido por seu equilíbrio entre tamanho compacto (1,1B parâmetros) e capacidade de gerar respostas coerentes. Modelos menores carecem de nuance semântica para capturar ironia de forma consistente, enquanto modelos maiores (7B+) tornariam o treinamento inviável na GPU do Kaggle.

A adaptação é feita via **LoRA (Low-Rank Adaptation)**, cujo princípio é decompor a atualização de peso de uma camada em duas matrizes de posto baixo: `W' = W + BA`, onde `B ∈ ℝ^{d×r}` e `A ∈ ℝ^{r×k}`, com `r ≪ min(d, k)`. Durante o treino, apenas `A` e `B` são atualizadas, de forma que os pesos originais `W` permanecem congelados. Isso significa que o modelo não reescreve o que já sabe; ele aprende um "desvio" de comportamento de baixa dimensão. As camadas alvo são `q_proj`, `k_proj`, `v_proj` e `o_proj`, pois o mecanismo de atenção é onde o modelo codifica padrões de estilo e tom.

## Inferências de Teste

**Prompt 1 — Busão lotado de manhã cedo:**

> *"How come you don't enjoy waking up early to take a crowded public bus?"*

**Resposta do modelo:**
> *"You do when it means sitting in front of your computer all day long."*

Clássico. O modelo aprendeu que a alternativa ao busão lotado é ficar na frente do computador.

---

**Prompt 2 — Celular no trabalho:**

> *"Why do people spend hours on their phones instead of working?"*

**Resposta do modelo:**
> *"The most productive way is in front of a real book! Do you ever use your phone while reading? I'm constantly doing that 😒 it's so funny because my friends also talk too long. We could all learn some patience."*

O modelo começou sarcástico, recomendou um livro, depois admitiu que fica no celular o tempo todo

---

**Prompt 3 — Palmeiras e o Mundial:**

> *"What do you think about Palmeiras, a Brazilian soccer club that has never won the Club World Cup?"*

**Resposta do modelo:**
> *"The only thing Palmieras 'win' is their name. They haven't even managed an appearance in FIFA Competitions!"*

O modelo errou o nome (escreveu *Palmieras*, demonstrando tamanha relevância do time). Apesar disso, citou times como Shenhua da China e Kuban Krasnodar da Rússia, ou seja, na visão do modelo, o Palmeiras está no mesmo nível de clubes que a maioria das pessoas precisaria pesquisar.