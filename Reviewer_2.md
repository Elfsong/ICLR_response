# Reviewer 2
Thank you for your valuable review and constructive suggestions!  I hope the response can address all the raised concerns and questions.

**Q1:** *Given longer enough context, the autoregressive model will ignore the effects of the latent variables, especially with powerful enough decoders. I see you use the cyclic annealing in Bert+GPT2 experiment to address this issue. However, I want to know more results and evaluation for more powerful models such as LLaMA-2.*

**Q2:** *Do you consider using learnable prior in the training rather than in a two-step manner?*

**Q3:** *Latent space EBM.*
We thank you for pointing out several important references that were missing in the original submission. We will carefully incorporate them in the final revision paper.

**Q4:** *For the ODE sampler, how does the ODE solver compare performance-wise to using Equ.11 directly for Langevin sampling (without noise)? Could you distinguish between your Runge-Kutta method and Langevin sampling in implementation?*

It's a good question. Attached is the comparison between Ordinary Differential Equation (ODE) and Stochastic gradient Langevin dynamics (SGLD) (without noise): 

The vanilla SGLD samples from an expected probability distribution $p(x)$ by the score function $\nabla_{x} \log(p(x))$. Given a fixed step size $\epsilon > 0$, a stochastic term $z_{t} \sim \mathcal{N}(0, I)$, and an initial value $x_0$ from a prior distribution, the Langevin method recursively computes the following:

$x_{t+1} = x_{t} + \underbrace{\frac{\epsilon}{2} \nabla_{x} \log(x_{t})}_{-E(x)} + \sqrt{\epsilon} z_{t}$

- [Diversity] The latent variable is envisioned as a low-dimensional manifold embedded within a higher-dimensional latent space. Consequently, most points from Langevin Sampling (without noise) won't align with the manifold. This implies that the likelihood of these points will be zero, rendering the score function $\log p(x)$ undefined for these points. Hence, SGLD tends to easily settle into local optima. In our work, cyclic annealing was leveraged to introduce scaled noise to the sample data. As a result, the generated latent variable is not only more robust but also converges more quickly.

- [Speed] Similar to the first problem, LD without noise struggles to garner enough sample data to effectively guide the score function in areas of sparsity. Consider the energy distributions depicted in Figure 6, where most samples are tightly clustered leaving other areas distinctly sparse. LD might employ a smaller learning rate alongside an increased number of learning steps to work around this problem, however, this would inevitably compromise the speed. The DICE method proposed in our paper manages this issue by initiating scale disturbance as well. The Runge-Kutta method is a deterministic process, in contrast to Langevin sampling which necessitates MCMC sampling, which samples from the approximated distribution for every gradient update. Therefore, the Runge-Kutta method proves to be more efficient than Langevin sampling in this case.

- [Joint Modelling] In our empirical experiments, we found both SGLD and ODE could generate fluent completions with desired attributes for a single-bias control, while SGLD is slower and less diverse. However, the performance of SGLD drastically deteriorates under joint debiasing settings. This decrease in performance might be attributed to SGLD's naive disregarding of the weight each joint score function carries. For example, consider a joint score function $ \log p(x) = \log (w_1 * p_1(x) + w_2 * p_2(x)) $, SGLD employs $ \nabla_x \log p(x) = \nabla_x \log p_1(x) + \nabla_x \log p_2(x) $ to sample from $p(x)$ which does not rely on these weights. 

- [Optimization] The SGLD approach is highly sensitive to hyperparameters, often requiring substantial time for optimization without guaranteeing fluent completion (see the empirical experiment). On the other hand, the Runge-Kutta method exhibits consistent performance across all tested LLMs(GPT2-base, GPT2-large, Llama-7b, and Llama2-7b) using the same hyperparameters. This trait makes it much more practical and adaptable in real scenarios.
