# Reviewer 4

**Q1:** Intrinsic and Extrinsic Evaluation

**Q2:** for the intuition part of the paper, why does removing the word 'Ethiopian' then the bias is reduced? how is the bias defined here?

**Q3:**  When using the BERT as encoder, do you use the [cls] vector as the latent space point?

We added an MLP layer on the top of BERT [cls]. We also explored the pooling of the sequence representation, the performance was similar.
