# Perturbed-Attention Guidance
<a href="https://arxiv.org/abs/2403.17377"><img src="https://img.shields.io/badge/arXiv-2403.17377-B31B1B"></a>
<a href="https://ku-cvlab.github.io/Perturbed-Attention-Guidance"><img src="https://img.shields.io/badge/Project%20Page-online-brightgreen"></a>
<a href="https://huggingface.co/hyoungwoncho/sd_perturbed_attention_guidance"><img src="https://img.shields.io/badge/HuggingFace-demo-yellow"></a>
<a target="_blank" href="https://colab.research.google.com/#fileId=https://huggingface.co/hyoungwoncho/sd_perturbed_attention_guidance/blob/main/sd_pag_demo.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

This is the official implementation of the paper "Self-Rectifying Diffusion Sampling with Perturbed-Attention Guidance",

by [Donghoon Ahn](https://github.com/sunovivid)\*, [Hyoungwon Cho](https://github.com/HyoungwonCho)\*, [Jaewon Min](https://github.com/Min-Jaewon), [Wooseok Jang](https://github.com/woo1726), [Jungwoo Kim](https://github.com/kmjnwn), [Seonhwa Kim](https://github.com/shhh0620), Hyunhee Park, [Kyonghwan Jin](https://ipa.korea.ac.kr)<sup>†</sup>, [Seungryong Kim](https://cvlab.korea.ac.kr)<sup>†</sup>.

![teaser](./imgs/teaser.png)

**Perturbed-Attention Guidance** significantly enhances the sample quality of diffusion models *without requiring external conditions*, such as class labels or text prompts, or *additional training*. This proves particularly valuable in unconditional generation settings, where classifier-free guidance (CFG) is inapplicable. Our guidance can be utilized to enhance performance in various downstream tasks that leverage unconditional diffusion models, including ControlNet with an empty prompt and image restoration tasks like super-resolution and inpainting.

For more information, check out [the project page](https://ku-cvlab.github.io/Perturbed-Attention-Guidance) and [the paper](https://arxiv.org/abs/2403.17377).

## News

**2024-03-31:** The [SD WebUI (automatic1111) extension](https://github.com/v0xie/sd-webui-incantations) is now available, thanks to the awesome work of @v0xie!

**2024-04-07:** The [application to **PSLD**](https://github.com/Min-Jaewon/PSLD_PAG) is now available!

**2024-04-12:** The [**ComfyUI / SD WebUI Forge node**](https://github.com/pamparamm/sd-perturbed-attention) is now available, thanks to the awesome work of @pamparamm!

## Overview

This repository is based on [SusungHong/Self-Attention-Guidance](https://github.com/SusungHong/Self-Attention-Guidance), which is based on [openai/guided-diffusion](https://github.com/openai/guided-diffusion). The environment setup and the pretrained models are the same as the original repository. The main difference is that the sampling code is modified to support perturbed-attention guidance. Please refer to [Using PAG in Guided-Diffusion](#Using-PAG-in-Guided-Diffusion) for environment setup and sampling.

If you're interested in utilizing PAG with Stable Diffusion, we have made available a [🤗🧨diffusers community pipeline](https://huggingface.co/hyoungwoncho/sd_perturbed_attention_guidance) on the HuggingFace Hub. There's no need to download the entire source code; simply specifying the `custom_pipeline` argument to **hyoungwoncho/sd_perturbed_attention_guidance** with the latest diffusers library (v0.27) is all that's required. Example code is provided in `sd_pag_demo.ipynb`.

## Using PAG with Stable Diffusion

You can try a demo in Colab! <a target="_blank" href="https://colab.research.google.com/#fileId=https://huggingface.co/hyoungwoncho/sd_perturbed_attention_guidance/blob/main/sd_pag_demo.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

Loading Custom Pipeline
```
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    custom_pipeline="hyoungwoncho/sd_perturbed_attention_guidance",
    torch_dtype=torch.float16
)

device="cuda"
pipe = pipe.to(device)
```
Sampling with PAG
```
output = pipe(
        prompts,
        width=512,
        height=512,
        num_inference_steps=50,
        guidance_scale=0.0,
        pag_scale=5.0,
        pag_applied_layers_index=['m0']
    ).images[0]
```
Sampling with PAG and CFG
```
output = pipe(
        prompts,
        width=512,
        height=512,
        num_inference_steps=50,
        guidance_scale=4.0,
        pag_scale=3.0,
        pag_applied_layers_index=['m0']
    ).images[0]
```

## Using PAG with Guided-Diffusion 
### Environment
The following commands are for setting up the environment using conda. 
- Python 3.9
- PyTorch 1.11.0, Torchvision 0.12.0
- NVIDIA RTX 3090
```
conda create -n pag python=3.9
conda activate pag

conda install gxx_linux-64 #
pip install torch==1.11.0+cu113 torchvision==0.12.0+cu113 -f https://download.pytorch.org/whl/torch_stable.html
pip install -r requirements.txt
```
If you have any issues with the environment setup, please refer to the original repository or [create an issue](https://github.com/sunovivid/Perturbed-Attention-Guidance/issues). We will gladly check it.

### Downloading Pretrained Diffusion Models (and Classifiers for CG)
Pretrained weights for ImageNet can be downloaded from [the repository](https://github.com/openai/guided-diffusion). Download and place them in the `./models/` directory.

### Sampling from Pretrained Diffusion Models
Run the baseline sampling code first to check if the environment is set up correctly.

### ImageNet 256 Unconditional Sampling (DDIM-25)

```
sh run/sample_uncond_ddim25_baseline.sh
```

The sampling code is modified to support perturbed-attention guidance. The following command samples from the pretrained model.


### ImageNet 256 Unconditional Sampling (DDPM-250)
```
sh run/sample_uncond_ddpm250.sh
```
### ImageNet 256 Conditional Sampling (DDPM-250)
```
sh run/sample_cond_ddpm250.sh
```

### ImageNet 256 Unconditional Sampling (DDIM-25)
```
sh run/sample_uncond_ddim25.sh
```
### ImageNet 256 Conditional Sampling (DDIM-25)
```
sh run/sample_cond_ddim25.sh
```

### Multi-GPU Sampling
If mpiexec is installed, you can use the following command to sample from multiple GPUs.

```
sh run/sample_uncond_ddim25@multigpu.sh
```
it is same with `run/sample_uncond_ddim25.sh` except for the following part.

```
GPU_COUNT=8 # number of GPUs to use
export NCCL_P2P_DISABLE=1 # for multi-node sampling
mpiexec -n $GPU_COUNT 
    ~ same code ~
    --gpu_offset 0  # change --gpu to --gpu_offset
    ~ same code ~
```

## PAG with Downstream tasks

Implementations applying PAG to downstream tasks are provided here.

### PSLD + PAG

Thanks to [LituRout/PSLD](https://github.com/LituRout/PSLD), we applied PAG to PSLD based on the repository.

PSLD is a framework to solve linear inverse problems leveraging pre-trained latent diffusion models. Please refer to [Min-Jaewon/PSLD_PAG](https://github.com/Min-Jaewon/PSLD_PAG) to use PSLD with PAG.

### Stable Diffusion Inpainting + PAG

Will be added soon.

## Community Implementation for GUI Interfaces (SD WebUI, WebUI Forge, and ComfyUI)

Thanks to the exceptional efforts of @v0xie and @pamparamm, you can now easily incorporate PAG into your pipelines or workflows.

**WebUI (Automatic1111)**: [v0xie/sd-webui-incantations](https://github.com/v0xie/sd-webui-incantations)

**ComfyUI / SD WebUI Forge**: [pamparamm/sd-perturbed-attention](https://github.com/pamparamm/sd-perturbed-attention)

# Citation
If you find our work useful in your research, please cite our work as:
```
@article{ahn2024self,
  title={Self-Rectifying Diffusion Sampling with Perturbed-Attention Guidance},
  author={Ahn, Donghoon and Cho, Hyoungwon and Min, Jaewon and Jang, Wooseok and Kim, Jungwoo and Kim, SeonHwa and Park, Hyun Hee and Jin, Kyong Hwan and Kim, Seungryong},
  journal={arXiv preprint arXiv:2403.17377},
  year={2024}
}
```
