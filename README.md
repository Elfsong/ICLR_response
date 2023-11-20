# Reviewer 1
We greatly appreciate your valuable feedback! It is our sincere hope that our responses can effectively address your concerns.

**Q1:** More details about the AtM and PSA are needed.

Both AtM [1] and PSA [2] operate as the infusion mechanism connecting the encoder and decoder.

AtM projects the latent variable z into both the attention key and value spaces by a MLP. It then concatenates them with the key and value vector in each attention layer within the decoder. This process can be thought of as the insertion of a "soft" prompt prefix ahead of the context. In this scenario, the latent variable functions as Q, with the Key and Value (KV) being projected by Q:

$k' = v' = mlp(z)$

$q = [q;z] \: k = [k;k'] \: v = [v;v']$

PSA is similar to AtM, except it has different K and V:

$k' = mlp_{k}(z) \: v' = mlp_{v}(z)$

$q = [q;z] \: k = [k;k'] \:v = [v;v']$

**Q2:** What are the labels in Algorithm 1 line 9?
The labels in Algorithm 1 line 9 can be found in Table 9. 

Algorithm 1 represents the DICE training and inference procedure, whereas Crows-Pairs / StereoSet benchmarks are denoted as the evaluation procedure. Labels bear different meanings within these two procedures.

Within the context of Algorithm 1, labels can guide DICE to produce completions with various bias inclinations. For instance, in a race-bias context, the label "Black" might prompt DICE to produce completions that have a bias towards black-racism. Conversely, the label "Neutral" may direct DICE to yield completions with less racial bias.

In the Crows-Pairs Benchmark, the labels are defined as {more, less}, and the bias direction is categorized as {stereotypical, anti-stereotypical}. For instance, For example, the statement "Blacks never listen to their parents" is deemed more stereotypical compared to "Whites never listen to their parents". We propose that racial identifiers (“Blacks” and “Whites”) contribute to the formation of bias. Consequently, we employ the label "Neutral" in Algorithm 1 to generate a more "neutral" latent variable, and therefore mitigate the race bias in LLM completions.

**Q3:** How to generate debiased context? Are you assigning labels as well? If so, how to guarentee that is not biased?

We don't assign a label to the context directly. The difference between our work and [3] here is that we encode the context to a latent variable and push it ahead to the "neutral" area in the latent space via an EBM-guided ODE solver. Then integrating the latent variable with the context by an infusion mechanism (AtM or PSA). As we have pretrained the encoder (BERT) and the decoder (LLM) together, the decoder can effectively interpret the latent variable's meaning, resulting in less biased completions. Conceptually, the latent variable serves as a "soft prompt" that is tailored to each individual context and guides the decoder to yield less biased completions. The baseline Self-Debias [4] has demonstrated the feasibility of using prompts to guide LLM debiasing, although "fixed hard prompts" were used in this baseline.

**Q4:** Confusing metric scores.
Sorry for the confusion caused. In the original table, the absolute difference between the raw ss score and 50 was utilized, denoted numerically as $ss = abs(50 - raw ss)$, so that the value should be the lower the better. This modification was unfortunately not explicitly conveyed.

In the revised paper, the raw ss score has been reinstated to maintain consistency with the initial definition. The ss score now serves as an indicator of the model's bias polarity. A score exceeding 50 represents stereotypical bias, while a score below 50 indicates anti-stereotypical bias. The ideal ss score is thus 50.

Regarding the GPT2-large Race score, a mistake occurred during the transformation of our results into the Latex table format. This issue has been rectified in the revised version.

| Model                   | Stereotype Score  |       |          | Language Modeling |       |          | StereoSet ICAT |       |          |
|-------------------------|-------------------|-------|----------|-------------------|-------|----------|----------------|-------|----------|
|                         | Gender            | Race  | Religion | Gender            | Race  | Religion | Gender         | Race  | Religion |
| GPT                     | 62.65             | 58.9  | 63.26    | 92.01             | 90.95 | 91.21    | 68.73          | 74.76 | 67.02    |
| GPT-2 + CDA             | 64.02             | 57.31 | 63.55    | 90.97             | 89.34 | 91.01    | 65.46          | 76.28 | 66.35    |
| GPT-2 + INLP            | 60.17             | 58.96 | 63.95    | 90.63             | 91.02 | 91.16    | 72.20          | 74.71 | 65.73    |
| GPT-2 + Self-Debias     | 60.84             | 57.33 | 60.45    | 90.41             | 89.4  | 89.65    | 70.81          | 76.29 | 70.91    |
| GPT + DICE              | 56.75             | 57.45 | 59.41    | 91.83             | 88.76 | 90.9     | 79.43          | 75.53 | 73.79    |
| GPT-Large               | 67.64             | 62.35 | 66.35    | 92.92             | 92.41 | 93.69    | 60.14          | 69.58 | 63.05    |
| GPT-Large + Self-Debias | 63.39             | 66.64 | 64.53    | 89                | 88.82 | 89.86    | 65.17          | 59.26 | 63.75    |
| GPT-Large + UDDIA-b     | 60.69             | 64    | -        | 88.07             | 87.59 | -        | 69.24          | 63.06 | -        |
| GPT-Large + DICE        | 61.05             | 64.24 | 65.71    | 90.41             | 90.13 | 91.25    | 70.43          | 64.46 | 62.58    |
| Llama                   | 69.3              | 67.01 | 61.04    | 92.64             | 92.27 | 93.1     | 56.88          | 60.88 | 72.54    |
| Llama + CDA             | 69.3              | 65.42 | 63.12    | 92.04             | 91.04 | 91.01    | 56.51          | 62.96 | 67.13    |
| Llama + INLP            | 67.51             | 66.43 | 65.24    | 89.18             | 90.57 | 89.92    | 57.95          | 60.81 | 62.51    |
| Llama+ Self-Debias      | 62.48             | 58.19 | 60.1     | 91.4              | 90.91 | 92.31    | 68.59          | 76.02 | 73.66    |
| Llama + DICE            | 59.53             | 43.91 | 58.1     | 91.83             | 90.77 | 91.43    | 74.33          | 79.71 | 76.62    |
| Llama 2                 | 66.27             | 64.06 | 60.41    | 88.83             | 88.83 | 92.27    | 59.92          | 63.85 | 73.06    |
| Llama 2 + CDA           | 64.03             | 67.24 | 60.19    | 86.42             | 89.02 | 90.41    | 62.17          | 58.33 | 71.98    |
| Llama 2 + INLP          | 63.97             | 62.5  | 60.33    | 85.41             | 90.04 | 88.98    | 61.55          | 67.53 | 70.60    |
| Llama 2 + Self-Debias   | 60.04             | 63.49 | 59.1     | 89.3              | 91.3  | 90.17    | 71.37          | 66.67 | 73.76    |
| Llama 2 + DICE          | 58.83             | 60.42 | 59.98    | 90.44             | 89.2  | 91.47    | 74.47          | 70.61 | 73.21    |


[1] Li, Chunyuan, et al. "Optimus: Organizing sentences via pre-trained modeling of a latent space." arXiv preprint _arXiv:2004.04092_ (2020).
[2] Fang, Le, et al. "Transformer-based conditional variational autoencoder for controllable story generation." _arXiv preprint arXiv:2101.00828_ (2021).
[3] Liu, Guangyi, et al. "Composable text controls in latent space with odes." _arXiv preprint arXiv:2208.00638_ (2022).
[4] Schick, Timo, Sahana Udupa, and Hinrich Schütze. "Self-diagnosis and self-debiasing: A proposal for reducing corpus-based bias in nlp." _Transactions of the Association for Computational Linguistics_ 9 (2021): 1408-1424.
