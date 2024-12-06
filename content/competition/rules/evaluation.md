---
title: Evaluation
type: docs
prev: competition/rules/submission
next: competition/prizes
weight: 3
math: true
---

The submitted prompts will undergo an evaluation process that involves subjecting them to 10 trials for each of the 26 uppercase letters of the English alphabet (A-Z). The levels generated for each character will be evaluated based on their similarity, stability, and diversity, and scored using the criteria outlined in the scoring policy given below. The entire evaluation process will be conducted using [automated scripts and programs](/competition/resources).

Please note that the evaluation process will be conducted twice, at midterm and final stages. The number of trials and characters in the evaluation set may be adjusted based on the number of teams.

## Evaluation Set

All 26 alphabetical uppercase characters.

<!-- A B C D E F G H I J K L M N O P Q R S T U V W X Y Z -->

$A \ B \ C \ D \ E \ F \ G \ H \ I \ J \ K \ L \ M \ N \ O \ P \ Q \ R \ S \ T \ U \ V \ W \ X \ Y \ Z$

## Scoring Policy

{{< callout type="error" >}}
Calculate the sum of scores for all three models
{{< /callout >}}

In trial $i$ of target character $j$ for program $k$:

1. **Stability**: The stability of a level is evaluated
   using our [Science Birds Evaluator](https://github.com/chatgpt4pcg/modified-science-birds). Here, $total\\_blocks\_{ijk}$ is defined as
   the number of blocks at the initialization step of loading the
   level into the evaluator. Then the program will calculate the
   value of $moving\\_blocks\_{ijk}$
   which is defined as the number of moving blocks during the first
   10 seconds after level initialization. Each level will receive an $sta\_{ijk}$ score according to the following
   equation. The score has a continuous value in $[0, 1]$.

   $$ sta\_{ijk} = \frac{total\\\_blocks\_{ijk} - moving\\\_blocks\_{ijk}}{total\\\_blocks\_{ijk}} $$

2. **Similarity**: The similarity score reflects the softmax probability $\sigma (\textbf{z}_{ijk})_j$
   of the model called [vit-base-uppercase-english-characters](https://huggingface.co/pittawat/vit-base-uppercase-english-characters),
   which is used to infer target character $j$ in
   trial $i$ from the image of the generated
   level after the first 10 seconds of initialization. Each level
   will receive a continuous value between $[0, 1]$. This score, $sim\_{ijk}$, is given as

   $$sim_{ijk} = \sigma (\textbf{z}_{ijk})_j$$

3. **Diversity**: The diversity score represents how
   diverse the generated levels are under the same target character $j$ for program $k$. The
   score, $div_{jk}$, is calculated by computing
   cosine distance, $D_c$, of unordered pairs
   in the set $A$ containing pairs of output
   vectors from the softmax probability, $\textbf{v}\_{ijk} = \sigma(\textbf{z}_{ijk})$.
   $\Xi\_{jk}$ denotes a set containing all such
   vectors across trials of the same target character $j$ from the same program $k$.

   $$div_{jk} = \frac{\sum_a^{|A_{jk}|} D_c(\textbf{v}_a^1, \textbf{v}_a^2)}{0.5T(T+1)-T}\text{,}$$

   where

   $$ A\_{jk} = \\{(\textbf{v}\_a^1, \textbf{v}\_a^2) | (\textbf{v}\_a^1, \textbf{v}\_a^2) \in \Xi\_{jk} \bowtie \Xi\_{jk} \land v_a^1 \neq v_a^2 \\} $$

   and $T$ represents the number of trials per target character.

Given that $P$ and $C$ represent the number of programs in the competition and the number of
characters, respectively, to give a higher weight to a more
difficult target character, $weight_{j}$ for
target character $j$ is defined as follows:

$$weight_{j} = w\\_sta\_{j} \times w\\_sim\_{j} \times w\\_div\_{j}\text{,}$$

where

$$w\\_sta\_{j} = max(1 - \frac{\sum\_{k=1}^{P} \sum\_{i=1}^{T} sta\_{ijk}}{PT}, \frac{1}{C})\text{,}$$

$$w\\_sim\_{j} = max(1 - \frac{\sum\_{k=1}^{P} \sum\_{i=1}^{T} sim\_{ijk}}{PT}, \frac{1}{C})\text{,}$$

and

$$w\\_div\_{j} = max(1 - \frac{\sum\_{k=1}^{P} div\_{jk}}{P}, \frac{1}{C})$$

Next, the weighted $trial_{ijk}$ score is defined as follows:

$$trial_{ijk} = weight_{j} \times sta_{ijk} \times sim_{ijk}$$

The average score for target character <InlineMath math='j' /> of
program $k$, $char_{jk}$, is defined as follows:

$$char_{jk} = \frac{div_{jk}\sum_{i=1}^{T} trial_{ijk}}{T}$$

The $prompt_{k}$ score is defined as follows:

$$prompt_{k} = \frac{\sum_{j=1}^{C} char_{jk}}{C}$$

Next, the normalized $prompt_{k}$ score, $norm\\_prompt\_{k}$, is defined as follows:

$$norm\\_prompt\_{k} = 100\ \frac{prompt\_{k}}{competition}\text{,}$$

where

$$competition = \sum_{k=1}^{P} prompt_{k}$$

Finally, $norm\\_prompt\_{k}$ will be used for ranking.

## Ranking Policy

The team that has the highest $norm\\_prompt\_{k}$ will be declared the winner. If there are multiple teams with the same highest score, the one with the shortest prompt will be chosen
as the winner. However, if multiple teams still have the same score
and the shortest prompt, they will be considered co-winners.

## Evaluation Tools

1. We will conduct the final evaluation on these 3 models:

   - `qwen2.5-32B`: `Q8_0` [bartowski/Qwen2.5-32B-Instruct-GGUF](https://model.lmstudio.ai/download/bartowski/Qwen2.5-32B-Instruct-GGUF)
   - `phi-3-medium`: `Q8_0` [ssmits/Phi-3-medium-128k-instruct-Q8_0-GGUF](https://model.lmstudio.ai/download/ssmits/Phi-3-medium-128k-instruct-Q8_0-GGUF)
   - `llama3.1-70B`: `Q4_K_M` [bartowski/Meta-Llama-3.1-70B-Instruct-GGUF](https://model.lmstudio.ai/download/bartowski/Meta-Llama-3.1-70B-Instruct-GGUF)

   While participants are free to explore other models during their research phase, our final assessment will be a comprehensive evaluation of submitted projects based on these three models.

2. [Science Birds Evaluator](https://github.com/chatgpt4pcg/modified-science-birds) with features to:

   - Automatically assess the stability.
   - Automatically export images using black-textured blocks on a white background for similarity testing.

3. [vit-base-uppercase-english-characters](https://huggingface.co/pittawat/vit-base-uppercase-english-characters).

4. Automation scripts available on our [Resources](/competition/resources) page.

## Evaluation Environment for Response Generation Stage

Software:

- OS: macOS Sonoma Version 14.5
- Python: 3.11.xx
- Unity: 2019.4.40f1 - [Science Birds Evaluator](https://github.com/chatgpt4pcg/modified-science-birds)
- [Our automation scripts](/competition/resources)

Hardware:

- Chip: Apple M2 Ultra - RAM: 128 GB

## Evaluation Process

1. We manually inspect each submitted program for a potential violation of the rules.
2. Submitted PE will be evaluated on all three models (average score)
3. Better PE performs well on all models
4. We will test generalizable of prompt engineering
5. The [code extraction script](https://github.com/chatgpt4pcg/code-extraction-script) will load each response and produce a new file containing only a series of `drop_block()` commands.
6. The [text-to-xml conversion script](https://github.com/chatgpt4pcg/text-to-xml-converter-script) will load each code file and convert it into a Science Birds level description XML file.
7. Next, [Science Birds Evaluator](https://github.com/chatgpt4pcg/modified-science-birds) will individually load all levels to assess their stability and capture their images. The results of stability will be recorded, and for each level an image of the structure with black-textured blocks on a white background will be produced by the program.
8. The [similarity checking script](https://github.com/chatgpt4pcg/similarity-checking-script) will load each image and pass it through an open source-model called [vit-base-uppercase-english-characters](https://huggingface.co/pittawat/vit-base-uppercase-english-characters). It will then record the similarity result.
9. The [diversity checking script](https://github.com/chatgpt4pcg/diversity-checking-script) assesses the diversity of the levels by averaging the cosine distance of non-duplicated all-pairs of outputs from softmax for each level across trial of the same target character as described in the [Evaluation](/competition/rules/evaluation).
10. The [scoring and ranking script](https://github.com/chatgpt4pcg/scoring-and-ranking-script) will load all stability, similarity, and diversity results and produce the final rank and score result for all teams according to the scoring policy.
11. Finally, we will calculate the sum of scores for all three models. Evaluation process for results from each model is same as this year competition.
