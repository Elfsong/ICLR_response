# Reviewer 3

We thank the reviewer for their thoughtful feedback and hope the response can address the raised concerns and questions.

**Q1**: How is your sampling method connected to diffusion models?

We employ an ODE solver in this work, which can be regarded as a sort of ``deterministic'' diffusion model [1].

Our motivation is to use the ODE solver to gradually converge the latent variable obtained from the VAE encoder to the energy area we expect under the guidance of EBM. This time-variant process can be regarded as a reverse diffusion process. [1] stated the reverse of a diffusion process is also a diffusion process. Crucially, this reverse process satisfies a reverse-time SDE, which can be derived from the forward SDE given the score of the marginal probability densities as a function of time. Moreover, for all diffusion processes, there exists a corresponding deterministic process whose trajectories share the same marginal probability densities as the SDE. This deterministic process satisfies an ODE [2]. Above is the connection between our sampling method and diffusion models.

**Q2:** Why you have opted for ODE instead of Langevin dynamics?

Regarding why we opted for ODE instead of Langevin dynamics, here are reasons from these aspects:

- [Diversity] The latent variable is envisioned as a low-dimensional manifold embedded within a higher-dimensional latent space. Consequently, most points from Langevin Sampling (without noise) won't align with the manifold. This implies that the likelihood of these points will be zero, rendering the score function $\log p(x)$ undefined for these points. Hence, SGLD tends to easily settle into local optima. In our work, cyclic annealing was leveraged to introduce scaled noise to the sample data. As a result, the generated latent variable is not only more robust but also converges more quickly.

- [Speed] Similar to the first problem, LD without noise struggles to garner enough sample data to effectively guide the score function in areas of sparsity. Consider the energy distributions depicted in Figure 6, where most samples are tightly clustered leaving other areas distinctly sparse. LD might employ a smaller learning rate alongside an increased number of learning steps to work around this problem, however, this would inevitably compromise the speed. The DICE method proposed in our paper manages this issue by initiating scale disturbance as well. The Runge-Kutta method is a deterministic process, in contrast to Langevin sampling which necessitates MCMC sampling, which samples from the approximated distribution for every gradient update. Therefore, the Runge-Kutta method proves to be more efficient than Langevin sampling in this case.

- [Joint Modelling] In our empirical experiments, we found both SGLD and ODE could generate fluent completions with desired attributes for a single-bias control, while SGLD is slower and less diverse. However, the performance of SGLD drastically deteriorates under joint debiasing settings. This decrease in performance might be attributed to SGLD's naive disregarding of the weight each joint score function carries. For example, consider a joint score function $ \log p(x) = \log (w_1 * p_1(x) + w_2 * p_2(x)) $, SGLD employs $ \nabla_x \log p(x) = \nabla_x \log p_1(x) + \nabla_x \log p_2(x) $ to sample from $p(x)$ which does not rely on these weights. 

- [Optimization] The SGLD approach is highly sensitive to hyperparameters, often requiring substantial time for optimization without guaranteeing fluent completion (see the empirical experiment). On the other hand, the Runge-Kutta method exhibits consistent performance across all tested LLMs(GPT2-base, GPT2-large, Llama-7b, and Llama2-7b) using the same hyperparameters. This trait makes it much more practical and adaptable in real scenarios.

**Q3:** How equation 8 satisfies the continuity equation and converges to your target distribution?

The ODE solver in our paper uses the Runge-Kutta of order 5 of Dormand-Prince-Shampine [3], which accepts any callable implementing the ordinary differential equation, even a neural network. Therefore, there is no strict continuity requirement to use the solver. We use Softplus instead of ReLU to avoid non-smooth non-linearities as well.

Convergence is guaranteed by the EBM-guided ODE solver, which depends on the quality of classifier training. In our case, the BERT encoder is enough to extract enough semantic information to allow the classifier to distinguish different biased attributes (Netrual F1: {"gender": 0.87, "race": 0.62, "religion": 0.47}).

**Q4:** How the generated latent variable guarantees the desired properties?

Our training procedure consists of two main steps:

Initially, we train the VAE on a pretraining corpus (wikitext-2) to establish the connection between the encoder and decoder. During this step, the latent variable produced by the encoder is directly fed into the decoder, skipping the ODE sampling.

Following that, we proceed to train the VAE on bias datasets. Here, the original latent variable is iteratively sampled to bear the desired bias attribute. This step ensures that the decoder focuses on the latent variable, thereby driving the decoder to generate completions with the desired properties.

**Q5:** How does the latent variable sampled from EBMs maintain consistency with the VAE latent space?

The answer to Q4 may answer this question.

[1] Anderson, Brian DO. "Reverse-time diffusion equation models." Stochastic Processes and their Applications 12.3 (1982): 313-326.

[2] Song, Yang, et al. "Score-based generative modeling through stochastic differential equations." arXiv preprint arXiv:2011.13456 (2020).

[3] Chen, Ricky TQ, et al. "Neural ordinary differential equations." Advances in neural information processing systems 31 (2018).


===

Thank you for your feedback. We are more than willing to answer your questions:

**Q1:**

Consider a diffusion process that iteratively blurs an image to reach a Gaussian distribution ($\mathcal{N}(0, I)$) in the forward process, and in reverse, iteratively samples an image back from a Gaussian distribution  ($\mathcal{N}(0, I)$). In this context, the time-varying probability $p_0$ depicts the image data distribution, and $p_T$ represents the initial distribution $\sim\mathcal{N}(0, I)$. An illustrative example can be found in Fig. 1 of [1]. Our model emulates the reverse diffusion process, which can be represented as solving an initial value problem (IVP) via ODE. We aim for the latent variable to gradually accommodate the desired attribute, enabling us to quantitively control the bias level. To this end, we employ an ODE solver (Runge-Kutta of order 5 of Dormand-Prince-Shampine) to approximate the target distribution given the original encoder output. Unlike conventional SDE which samples from a normal distribution, we treat each training data as a sample and then convert it into a representation in the latent space through the encoder. We can change the name if you feel it is inappropriate to call it sampling.

We hypothesize that our main divergence lies in how to sample from the latent space effectively. Though gradient descent seems an intuitive approach to approximate the target distribution, it exhibits significant limitations in our scenario. In our empirical experiments, we found that gradient descent is laborious to train and tends to produce unstable output—an issue also noted in [1,2,3]. Moreover, our model is designed to debias multiple attributes simultaneously, a scenario that SGLD may not adeptly accommodate. In fact, SGLD's performance significantly deteriorates under joint debiasing settings. This decrease in performance can potentially be attributed to the naive disregard by SGLD of the weight that each joint score function holds. For instance, in the case of a joint score function $ \log p(x) = \log (w_1 * p_1(x) + w_2 * p_2(x)) $, SGLD employs $ \nabla_x \log p(x) = \nabla_x \log p_1(x) + \nabla_x \log p_2(x) $ to sample from $p(x)$, without considering the corresponding weights.

**Q2:** 

**Slower**: When two modes of a given data distribution are divided by areas of low density, Langevin dynamics may not effectively recover the relative weights of these modes within a reasonable time, and thus may not converge to the true distribution [1]. This analysis also holds when different modes have approximately disjoint supports - they might share the same support but be linked via areas of low data density.

In a joint debiasing scenario, where biases related to "gender", "race", and "religion" are to be addressed simultaneously, the common support may not be adequate to recover the target distribution using SGLD. In theory, Langevin dynamics could indeed generate accurate samples, but this might necessitate a very small step size and a huge number of steps for effective mixing.

**Less diverse:** The latent variable is a low-dimensional manifold embedded within a higher-dimensional latent space. If we consider the vanilla SGLD without annealed noise, most points from Langevin sampling won't align with the manifold [1]. This implies that the likelihood of these points will be zero, rendering the score function undefined for these points. Hence, SGLD tends to easily settle into local optima and cluster together. To address this problem, we introduce scaled noise to perturb the supports, despite the ODE's deterministic nature. Consequently, our approach can generate a wider range of diverse results.

**Q3:** 

I agree with your point. There is indeed potential danger if the sample strays significantly away from the learning space. Current empirical observations indicate less impact on the decoder (likely attributable to the fewer than 10 ODE iterations performed). To further confirm these findings, we plan to conduct additional experiments, projecting parameters into the decoder’s constraint space, and providing theoretical boundaries for the latent variables. Your suggestions are greatly appreciated!

[1] Song, Yang, et al. "Score-based generative modeling through stochastic differential equations." _arXiv preprint arXiv:2011.13456_ (2020).

[2] Nie, Weili, Arash Vahdat, and Anima Anandkumar. "Controllable and compositional generation with latent-space energy-based models." _Advances in Neural Information Processing Systems_ 34 (2021): 13497-13510.

[3] Liu, Guangyi, et al. "Composable text controls in latent space with odes." _arXiv preprint arXiv:2208.00638_ (2022).

===

Thanks for your great effort in responding to my questions.

"To address this problem, we introduce scaled noise to perturb the supports, despite the ODE's deterministic nature. Consequently, our approach can generate a wider range of diverse results"

Could you explain briefly what you mean by "introduce scaled noise to perturb the supports"? Is there a corresponding description in the paper?

We appreciate your prompt reply!

The corresponding description can be found in Appendix 2.

$ \mathcal{L}_{vae}(x)=-E\_{q(z|x)}  [\log p(x|z)] + \beta \cdot \rm{KL} (q(z|x) || p\_{prior}(z))$

Briefly, this trick imports a coefficient $\beta$ that regulates the weight of the KL divergence on the VAE objective. The coefficient is gradually changed from 0 to 1. The motivation for doing this is as follows:

- Since the definition domain of Gaussian noise is the entire latent space, adding noise to the original data solves the zero probability problem of low-dimensional manifolds (otherwise the score function will not be able to steer the ODE solver).

- Adding noise essentially expands the range of each mode in the distribution, allowing the low-probability areas in the data distribution to get more supervision signals. In particular, this is very effective for the joint-debiasing scenario.

- The noise scale selection is not a simple problem. Adding too much noise will cover more low-probability space and drastically change the original data distribution, while too small noise will be useless to the low-probability space less dense. 

Therefore, the final solution we ended up using was to add a 4-loop annealing factor to rescale the KL divergence.


