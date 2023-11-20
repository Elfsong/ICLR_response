# Reviewer 3

We thank the reviewer for their thoughtful feedback and hope the response can address the raised concerns and questions.

**Q1**: How is your sampling method connected to diffusion models?

It's a good question. Thank you for questioning. To be concise, we employ an ODE solver in this paper, which can be regarded as a sort of ``deterministic'' diffusion model [1].

Our motivation is to use the ODE solver to gradually converge the latent variable obtained from the VAE encoder to the energy area we expect under the guidance of EBM. This time-variant process can be regarded as a reverse diffusion process. [1] stated the reverse of a diffusion process is also a diffusion process. Crucially, this reverse process satisfies a reverse-time SDE, which can be derived from the forward SDE given the score of the marginal probability densities as a function of time. Moreover, for all diffusion processes, there exists a corresponding deterministic process whose trajectories share the same marginal probability densities as the SDE. This deterministic process satisfies an ODE [2]. Above is the connection between our sampling method and diffusion models.

**Q2:** Why you have opted for ODE instead of Langevin dynamics?

Regarding why we opted for ODE instead of Langevin dynamics, here are reasons from these aspects:

- [Diversity] The latent variable is envisioned as a low-dimensional manifold embedded within a higher-dimensional latent space. Consequently, most points from Langevin Sampling (without noise) won't align with the manifold. This implies that the likelihood of these points will be zero, rendering the score function $\log p(x)$ undefined for these points. Hence, SGLD tends to easily settle into local optima. In our work, cyclic annealing was leveraged to introduce scaled noise to the sample data. As a result, the generated latent variable is not only more robust but also converges more quickly.

- [Speed] Similar to the first problem, LD without noise struggles to garner enough sample data to effectively guide the score function in areas of sparsity. Consider the energy distributions depicted in Figure 6, where most samples are tightly clustered leaving other areas distinctly sparse. LD might employ a smaller learning rate alongside an increased number of learning steps to work around this problem, however, this would inevitably compromise the speed. The DICE method proposed in our paper manages this issue by initiating scale disturbance as well. The Runge-Kutta method is a deterministic process, in contrast to Langevin sampling which necessitates MCMC sampling, which samples from the approximated distribution for every gradient update. Therefore, the Runge-Kutta method proves to be more efficient than Langevin sampling in this case.

- [Joint Modelling] In our empirical experiments, we found both SGLD and ODE could generate fluent completions with desired attributes for a single-bias control, while SGLD is slower and less diverse. However, the performance of SGLD drastically deteriorates under joint debiasing settings. This decrease in performance might be attributed to SGLD's naive disregarding of the weight each joint score function carries. For example, consider a joint score function $ \log p(x) = \log (w_1 * p_1(x) + w_2 * p_2(x)) $, SGLD employs $ \nabla_x \log p(x) = \nabla_x \log p_1(x) + \nabla_x \log p_2(x) $ to sample from $p(x)$ which does not rely on these weights. 

- [Optimization] The SGLD approach is highly sensitive to hyperparameters, often requiring substantial time for optimization without guaranteeing fluent completion (see the empirical experiment). On the other hand, the Runge-Kutta method exhibits consistent performance across all tested LLMs(GPT2-base, GPT2-large, Llama-7b, and Llama2-7b) using the same hyperparameters. This trait makes it much more practical and adaptable in real scenarios.

**Q3:** How equation 8 satisfies the continuity equation and converges to your target distribution?

We are using the Runge-Kutta of order 5 of Dormand-Prince-Shampine. 

**Q4:** How the generated latent variable guarantees the desired properties?

Pretrain + ODE Solver + Infusion

**Q5:** How does the latent variable sampled from EBMs maintain consistency with the VAE latent space?

Pretrain + ODE Solver + Decoder 

[1] Anderson, Brian DO. "Reverse-time diffusion equation models." Stochastic Processes and their Applications 12.3 (1982): 313-326.

[2] Song, Yang, et al. "Score-based generative modeling through stochastic differential equations." arXiv preprint arXiv:2011.13456 (2020).
