# Reviewer 4

We thank the reviewer for the comments and questions! We hope the response can address all the raised concerns and questions.

**Q1:** Intrinsic and Extrinsic Evaluation

We conduct extrinsic evaluations on three benchmarks shown below. All of them employed the sequence-to-sequence style. For the classification tasks, we limited the vocabulary list to ensure we could access the logit of each class. The results revealed that our method exceeded the performance of the majority of baselines within these extrinsic benchmarks and was successful in striking an optimal balance between maintaining language modeling capability while minimizing bias.

Regarding the intrinsic benchmark [1], which is designed for the masked language models. It is not compatible with the generative language models. Meanwhile, recent studies [2, 3] have brought the validity of SEAT into question due to the counter-intuitive results it often produces. As referenced in [4], SEAT is found to lack a consistent correlation with extrinsic metrics. This finding implies that a model can potentially receive high scores from SEAT, while simultaneously rendering biased judgments in downstream conditions. Therefore, we have chosen to disregard SEAT as an evaluation metric in our work.

#### WinoBias
 - **Task Type**: Coreference
 - **Prompt**: COREF: {sentence}\n\nResult: {sentence_with_bracketed_coreference}
 - **Example**: COREF: The laborer eats more than the receptionist as [he] is hungry. \n\nResult: [The laborer] eats more than the receptionist as [he] is hungry.
 - **Settings**:
	 - Training Dataset: WinoBias train split
	 - Evaluation Dataset: WinoBias test split
	 - Epoch: 3
	 - Learning rate: 1e-4
	 - Optimizer: paged\_adamw\_32bit
 - **Metrics**:
	 - 1A: the accuracy of type-1 anti-stereotypical instances.	
	 - 1P: the accuracy of type-1 stereotypical instances.		
	 - 2A: the accuracy of type-2 anti-stereotypical instances.
	 - 2P: the accuracy of type-3 stereotypical instances.
	 - TPR-1: the gap between 1P and 1A.
	 - TPR-2: the gap between 2P and 2A
 - **Results**: 

| Models                |    1A ↑   |    1P ↑   |    2A ↑   |    2P ↑   |      TPR-1 ↓      |      TPR-2 ↓      |
|-----------------------|:---------:|:---------:|:---------:|:---------:|:-----------------:|:-----------------:|
| Llama                 |     65.15 | **92.42** |     94.19 | **96.21** |             27.27 |              2.02 |
| Llama + CDA           |     61.11 |     85.35 |     82.83 |     81.06 |     24.24 (-3.03) |      1.77 (-0.25) |
| Llama + INLP          |     56.06 |      54.8 |     61.87 |     58.84 | **1.26 (-26.01)** |      3.03 (+1.01) |
| Llama + Self-Debias   |     71.21 |     91.67 |     94.95 |     92.93 |     20.46 (-6.81) |          2.02 (0) |
| Llama + DICE          | **72.47** |      89.9 | **95.45** |     95.97 |     17.43 (-9.84) |   **0.52 (-1.5)** |
| Llama 2               |     65.66 | **94.95** |     97.98 |     98.99 |             29.29 |              1.01 |
| Llama 2 + CDA         |      60.1 |      84.6 |     78.28 |     81.82 |      24.5 (-4.79) |      3.54 (+2.53) |
| Llama 2 + INLP        |     58.59 |     60.61 |     64.65 |     56.82 | **2.02 (-27.27)** |      7.83 (-6.82) |
| Llama 2 + Self-Debias |     69.95 |     91.41 |     97.98 |     97.98 |     21.46 (-7.83) |     **0 (-1.01)** |
| Llama 2 + DICE        | **72.11** |      93.5 | **98.74** | **99.24** |      21.39 (-7.9) |       0.5 (-0.51) |
| GPT 2                 |     47.12 |     52.03 |     56.17 | **71.72** |              4.91 |             15.55 |
| GPT 2 + CDA           |     46.46 |     48.74 |     57.83 |     69.44 |      2.28 (-2.63) |     11.61 (-3.94) |
| GPT 2 + INLP          |        50 |     48.48 |     54.04 |     51.01 |      1.52 (-3.39) | **3.03 (-12.52)** |
| GPT 2 + Self-Debias   |     50.76 |     49.75 |     57.58 |     51.77 |   **1.01 (-3.9)** |      5.81 (-9.74) |
| GPT2 + DICE           | **63.64** | **64.65** | **63.89** |     68.94 |   **1.01 (-3.9)** |      5.05 (-10.5) |

#### Bias-NLI
 - **Task Type**: Natural Language Inference
 - **Prompt**: Hypothesis: {hypothesis}\n\nPremise: {premise}\n\nResult: {label}
 - **Example**: Hypothesis: A person on a horse jumps over a broken down airplane.\n\nPremise: A person is outdoors, on a horse.\n\nResult: entailment
 - **Settings**:
	 - Training Dataset: SNLI dataset 
	 - Evaluation Dataset: Bias-NLI dataset
	 - Epoch: 3
	 - Learning rate: 1e-4 
	 - Optimizer: paged\_adamw\_32bit
 - **Metrics**: 
	 - Net Neutral (NN): The average probability of the neutral label across all sentence pairs.
	  - Fraction Neutral (FN): The fraction of sentence pairs labeled neutral.
	  - T: A parameterized measure that reports the fraction of examples whose probability of neutral above t: we report this for t = 0.5 and t = 0.7. 
- **Results**: 

| Models                | Acc. (All) ↑ | Acc. (M) ↑ | Acc. (F) ↑ | TPR-GAP ↓ | TPR-RMS ↓ |
|-----------------------|:------------:|:----------:|:----------:|:---------:|:---------:|
| Llama                 |        91.12 |  **91.73** |       90.4 |      1.33 |      0.09 |
| Llama + CDA           |        85.56 |      86.88 |      84.01 |      2.87 |      0.12 |
| Llama + INLP          |        85.53 |       84.5 |      86.73 |      2.23 |      0.11 |
| Llama + Self-Debias   |        91.22 |      90.33 |  **92.26** |      1.93 |       0.1 |
| Llama + DICE          |    **91.63** |      91.24 |      92.08 |  **0.84** |  **0.08** |
| Llama 2               |    **89.94** |   **90.6** |      89.17 |      1.43 |      0.09 |
| Llama 2 + CDA         |        82.89 |      84.33 |      81.21 |      3.12 |      0.14 |
| Llama 2 + INLP        |        87.01 |       87.5 |      86.44 |      1.06 |      0.08 |
| Llama 2 + Self-Debias |        89.66 |      89.27 |  **90.12** |      0.85 |  **0.04** |
| Llama 2 + DICE        |        89.21 |      89.06 |      89.39 |  **0.33** |  **0.04** |
| GPT 2                 |    **87.33** |      88.07 |  **86.46** |      1.61 |      0.15 |
| GPT 2 + CDA           |        84.68 |       87.3 |      81.62 |      5.68 |      0.23 |
| GPT 2 + INLP          |        84.81 |       85.5 |         84 |   **1.5** |  **0.13** |
| GPT 2 + Self-Debias   |        86.93 |      87.71 |      86.02 |      1.69 |      0.14 |
| GPT2 + DICE           |        87.30 |  **88.64** |      85.74 |       2.9 |      0.16 |

#### Bias-in-Bios
- **Task Type**: Classification
- **Prompt**: CLS: {biographies}\n\nResult: {profession}
- **Example**: CLS: Prior to law school, Brittni graduated magna cum laude from DePaul University in 2011...\n\nResult: 2(attorney)
- **Settings**:
	- Training Dataset: Bias-in-Bios train split
	- Evaluation Dataset: Bias-in-Bios test split
	- Epoch: 3
	- Learning rate: 1e-4
	- Optimizer: paged\_adamw\_32bit
- **Metrics**:
	- acc_a: the overview accuracy.
	- acc_m: the accuracy of male instances.
	- acc_f: the accuracy of female instances.
	- TPR-GAP: the gap between acc_m and acc_f.
	- TPR-RMS: Root-mean-square deviation between male and female predictions.
- **Results**:

| Models                |    NN ↑   |    FN ↑   |  T:0.5 ↑  |  T:0.7 ↑  |
|-----------------------|:---------:|:---------:|:---------:|:---------:|
| Llama                 |     73.85 |     96.63 | **98.46** |     92.19 |
| Llama + CDA           |      65.4 |        89 |      92.1 |     87.03 |
| Llama + INLP          |     69.08 |     92.58 |     96.43 |     89.04 |
| Llama + Self-Debias   | **85.22** |     96.97 |     97.66 |     91.26 |
| Llama + DICE          |     84.96 | **97.55** |     98.39 | **94.51** |
| Llama 2               |      76.4 |      95.9 |     98.17 |     90.93 |
| Llama 2 + CDA         |     70.11 |     91.07 |     90.01 |     81.15 |
| Llama 2 + INLP        |     75.24 |     90.11 |     88.92 |     80.09 |
| Llama 2 + Self-Debias |     85.89 | **98.01** |     97.39 |     92.53 |
| Llama 2 + DICE        | **87.71** |     97.66 |  **98.2** |  **94.2** |
| GPT 2                 |     79.01 | **90.13** |     87.66 |     74.35 |
| GPT 2 + CDA           |      77.3 |     89.71 |     85.91 | **76.76** |
| GPT 2 + INLP          |     72.46 |     85.98 |        82 |     74.06 |
| GPT 2 + Self-Debias   |     81.49 |     89.29 | **89.57** |     73.58 |
| GPT2 + DICE           | **82.66** |     90.04 |     88.02 |     73.77 |

**Q2:** for the intuition part of the paper, why does removing the word 'Ethiopian' then the bias is reduced? how is the bias defined here?

The Bias definition can be found in Section 2.1.

Let's take a sentence completion task as an example where an LLM is given a prompt such as "Many people live in Ethiopia". The resulting completions might reflect stereotypical views like "the people are very thin and proficient at long-distance running", or anti-stereotypical completions "the people are overweight and lack athletic abilities". To mitigate racial bias, it's essential for LLMs to avoid generating either stereotypical or anti-stereotypical responses. Ideally, the LLM should generate both types of completions with roughly equivalent probabilities. 

A common method employed to realize this involves removing or obfuscating race-specific information in the context (more specifically, alleviating the LLM attention on the race trigger word "Ethiopian". The ideal situation will be the LLM ignores bias trigger words, which equals to the prompt "Many people live on Earth" which has no race information).

Motivated by this, we propose the integration of a latent variable, representing the neutral bias attribute, into our decoder. This modification aims to guide the decoder towards generating less biased sentence completions.

**Q3:**  When using the BERT as encoder, do you use the [cls] vector as the latent space point?

We added an MLP layer on the top of BERT [cls]. We also explored the pooling of the sequence representation, the performance was similar.

[1] Kaneko, Masahiro, and Danushka Bollegala. "Unmasking the mask–evaluating social biases in masked language models." _Proceedings of the AAAI Conference on Artificial Intelligence_. Vol. 36. No. 11. 2022.

[2] May, Chandler, et al. "On measuring social biases in sentence encoders." _arXiv preprint arXiv:1903.10561_ (2019).

[3] He, Jacqueline, et al. "Mabel: Attenuating gender bias using textual entailment data." _arXiv preprint arXiv:2210.14975_ (2022).

[4] Goldfarb-Tarrant, Seraphina, et al. "Intrinsic bias metrics do not correlate with application bias." _arXiv preprint arXiv:2012.15859_ (2020).
