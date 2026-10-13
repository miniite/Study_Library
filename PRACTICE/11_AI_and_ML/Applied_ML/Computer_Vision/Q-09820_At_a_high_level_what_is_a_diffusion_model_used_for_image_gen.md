---
type: "practice"
difficulty: "Deep"
---

# At a high level, what is a diffusion model used for image generation and how does it iteratively denoise?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Image Normalization](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Image_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Diffusion models are a class of generative models that have gained popularity in recent years due to their ability to generate highly realistic images, video frames, and other multimodal data. At a high level, diffusion models iteratively denoise an image representation to generate new samples. The process starts from a random noise vector and gradually transforms it into a meaningful image through a series of steps.

To understand how this works, let's first consider the triage: the core idea is that we start with a noisy version of an image and apply a sequence of denoising transformations until we get a clean, generated image. The key assumptions include the use of stochastic processes to generate diverse samples and the ability to reverse these processes for efficient sampling.

Now let's break down the first-principles solution:

Model Architecture: A diffusion model consists of two main components: a denoising process that gradually removes noise from an input, and a learning process where we train the model to predict the correct noise at each step.
Training Process:
Start with a clean image ( x_0 ) and add noise through a stochastic process, resulting in a noisy version of the image ( x_t = x_0 + \epsilon ), where ( t ) is the timestep and ( \epsilon ) represents the added noise.
Train the model to predict the noise at each step given the current noisy state. This can be represented as learning the function ( q_\theta(x_{t-1} | x_t) ).
Sampling Process:
Start with a random noise vector and iteratively apply the learned denoising process.
At each step, predict the noise component from the current noisy image and subtract it to generate a cleaner image representation.

Here is a simplified step-by-step explanation of how this works:

Initialization: Begin with a random noise vector ( x_t ) (where ( t = T )).
Iterative Denoising:
For each time step ( t ) from ( T ) to 1, use the model to predict the noise component and subtract it.
The equation for denoising can be written as: ( x_{t-1} = \sqrt{1-\beta_t} x_t + \sqrt{\beta_t} \epsilon ), where ( \beta_t ) is a parameter controlling the amount of noise added at each step.

Now, let's consider a simple example to illustrate this:

PYTHON
import numpy as np

# Parameters
T = 500  # Number of time steps
beta = [0.1 * i for i in range(1, T + 1)]  # Noise parameter schedule

def denoise(x_t):
    noise_pred = np.random.normal(scale=np.sqrt(beta[t]), size=x_t.shape)
    return x_t - noise_pred

# Initialize with random noise
x_T = np.random.randn(64, 64)  # Example: 64x64 image

for t in range(T, 0, -1):
    x_T = denoise(x_T)

print("Generated Image:", x_T)

This code snippet demonstrates the iterative denoising process starting from a random noise vector and converging to a clean generated image.

To summarize, diffusion models iteratively denoise an initial noisy representation of an image until it resembles a clean and realistic sample. This approach is effective for generating diverse and high-quality images due to its principled probabilistic framework.

#### Common mistakes
include misunderstanding the stochastic nature of the process or failing to grasp the importance of the noise schedule in controlling the sampling process.

