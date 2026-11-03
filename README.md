[English](README.md) · [한국어](README.ko.md) · [中文](README.zh.md)

# image-generation-qwen-image-pruna

This service takes text and makes an image with the P-Q-Image-2.1 6-steps v0.2 LoRA on top of Qwen/Qwen-Image-2.1.

| Part | Weights |
| :---: | --- |
| Base model | [Qwen/Qwen-Image-2.1](https://huggingface.co/Qwen/Qwen-Image-2.1/tree/d26bb61231c349cf6b7896fa83353113880e1ba3) |
| LoRA | `p_q_image_2.1_6_steps_v0.2.safetensors` from [PrunaAI/P-Q-Image-2.1](https://huggingface.co/PrunaAI/P-Q-Image-2.1) |

![A black-and-white photo prompt goes into the web UI, and a black-and-white photo of a girl hugging a teddy bear on a swing comes out.](docs/images/demo.gif)

## Quick Start

It needs [model-compose](https://github.com/hanyeol/model-compose) 0.4.121 or later.

Install with [uv](https://docs.astral.sh/uv/):

```bash
uv pip install model-compose
```

Or with pip:

```bash
pip install model-compose
```

Clone this repository:

```bash
git clone https://github.com/MindrLabs/image-generation-qwen-image-pruna
cd image-generation-qwen-image-pruna
```

Run it:

```bash
model-compose up
```

The API listens on http://localhost:8080/api, and the Gradio UI opens at http://localhost:8081.

## Summary

On the DGX Spark, P-Q-Image-2.1 6-steps v0.2 took 10.9 seconds per image against 51.3 seconds for the base model at 40 steps, and a person judged that image quality did not drop noticeably.

| Machine | Status | Setting | Seconds per image (seed=0) | Peak memory (MiB, seed=0) |
| :---: | --- | --- | --- | --- |
| DGX Spark | Measured | P-Q-Image-2.1 6-steps v0.2 | 10.9 | 39907 (VRAM) |
| RTX 4090 | Measured | P-Q-Image-2.1 6-steps v0.2 | 46.4 | 17532 (VRAM) |
| MacBook M1 16GB | Measured | P-Q-Image-2.1 6-steps v0.2, NF4 512px | 115 | 15581 (footprint) |
| RTX 4050 Laptop | Not measured | — | — | — |

## Setup

| Setting | Inputs | Checkpoint | model-compose | Statistics |
| :---: | --- | --- | --- | --- |
| P-Q-Image-2.1 6-steps v0.2 | 3 inputs × 1 repeat | [Qwen/Qwen-Image-2.1](https://huggingface.co/Qwen/Qwen-Image-2.1/tree/d26bb61231c349cf6b7896fa83353113880e1ba3) | 0.4.121 | 95% intervals, speed differences under 1.05× are not called |
| Base model, 40 steps | 3 inputs × 1 repeat | [Qwen/Qwen-Image-2.1](https://huggingface.co/Qwen/Qwen-Image-2.1/tree/d26bb61231c349cf6b7896fa83353113880e1ba3) | 0.4.121 | 95% intervals, speed differences under 1.05× are not called |
| P-Q-Image-2.1 6-steps v0.2, NF4 512px | 3 inputs × 3 repeats | [Qwen/Qwen-Image-2.1](https://huggingface.co/Qwen/Qwen-Image-2.1/tree/d26bb61231c349cf6b7896fa83353113880e1ba3) | 0.4.121 | 95% intervals, speed differences under 1.05× are not called |

## Results

### Text to Image

The composition and pose of three black-and-white daily-life scenes of a teenage girl were written as English prompts and generated with three seeds.

| Claim | Check | Pass |
| :---: | --- | --- |
| A detailed prompt controls the subject, setting, lighting, and style of the image. | Judged by a person | ✓ |
| The 6-step adapter is meant as a drop-in replacement for the base model in generation workflows. | Judged by a person | ✓ |
| The v0.2 adapter lets Qwen-Image-2.1 generate a finished image in only 6 steps. | Judged by a person | ✓ |

Measured on the DGX Spark, the RTX 4090 and the MacBook M1 16GB; the RTX 4050 Laptop ran out of memory and was not measured. Each cell is the median over measured calls, leaving out any call made during other GPU work.

| Setting | Machine | Seconds per image (seed=0 / 1 / 2) | Peak memory (MiB) | How it ran |
| :---: | --- | --- | --- | --- |
| P-Q-Image-2.1 6-steps v0.2 | DGX Spark | 10.9 / 10.9 / 10.9 | 39907 (VRAM) |  |
| P-Q-Image-2.1 6-steps v0.2 | RTX 4090 | 46.4 / 46.5 / 46.5 | 17562 (VRAM) | Model CPU offload |
| Base model, 40 steps | DGX Spark | 51.3 / 51.2 / 51.3 | 39515 (VRAM) |  |
| Base model, 40 steps | RTX 4090 | 62.7 / 62.7 / 62.6 | 17530 (VRAM) | Model CPU offload |
| P-Q-Image-2.1 6-steps v0.2, NF4 512px | MacBook M1 16GB | 115 / 116 / 115 | 15581 (footprint) | NF4 quantization |

![bollard-night, RTX 4090](docs/images/text-to-image-bollard-night.jpg)

> Black-and-white candid daily-life photograph, photorealistic, at night under harsh flash. An Asian teenage girl in a school uniform — dark knit sweater, white collar, tie, pleated skirt, white slouch socks — sits slumped on the gravel ground, leaning back against a curved metal parking bollard, knees bent, head bowed so her black bob and bangs hide her face, utterly exhausted. Her school bag lies spilled open beside her with a small plush charm. Wooden fence and concrete curb behind, painted parking lines on the asphalt, flash-lit grainy snapshot, lonely late-night mood.

![cat-window, RTX 4090](docs/images/text-to-image-cat-window.jpg)

> Black-and-white candid daily-life photograph, photorealistic. An Asian teenage girl with a shoulder-length bob, seen from behind at a three-quarter angle in a dark jacket, standing below a ground-floor window and tilting her face up toward a black-and-white cat walking along the window ledge behind a white safety net. Rough plaster wall with faded graffiti, dusk light, soft film grain, quiet melancholic mood.

![swing-teddy, RTX 4090](docs/images/text-to-image-swing-teddy.jpg)

> Black-and-white candid daily-life photograph, photorealistic. An Asian teenage girl with a black bob and blunt bangs sits on a playground swing, hugging a large teddy bear to her chest with her cheek resting on its head, gazing off to the side. Oversized grey hoodie, plaid skirt, black backpack strap, swing chains in the foreground, empty playground and grass behind, overcast soft light, film grain, wistful mood.

## Recommendations

- Use P-Q-Image-2.1 6-steps v0.2 when speed matters. On the DGX Spark it took 10.9 seconds per image, shorter than the base model's 51.3 seconds.

## Limitations and What We Did Not Measure

### Measurement Conditions

| Where | Detail |
| :---: | --- |
| MacBook M1 16GB | The NF4 workflow ran on an Apple M1 with 16 GB of unified memory, where the whole pipeline stays in one memory pool. |
| GPU with separate VRAM | The NF4 pipeline is first loaded onto the GPU and model CPU offload then keeps the idle modules in system memory, so it needs enough of both. |
| NF4 loading | Loading reads the BF16 weights before quantizing them, which briefly needs more system memory than the quantized pipeline itself. So a laptop with a 6 GB GPU and 15 GB of system memory runs out of memory even though its total memory is close to the M1's. |
| NF4 comparison | The NF4 workflow has no comparison against the base model without the add-on. |
| RTX 4090 | The BF16 workflows run with model CPU offload because the pipeline does not fit in 24 GB of VRAM, so each request also spends time moving modules between system memory and the GPU. |
| DGX Spark | The workflows run with CPU offload turned off, because its memory is shared between the CPU and the GPU. |

### Not Measured

| What | Why |
| :---: | --- |
| RTX 4050 Laptop | The laptop's 15 GB of system memory ran out while the NF4 pipeline was offloaded to it, and the Linux environment stopped before any image was measured. |
| DGX Spark vs RTX 4090, seconds per image (seed=0, 1, 2) | 3 inputs were measured on both machines without other GPU work, and at least 20 are needed to compare them. |
| Lighting under prompt control | Lighting was not judged in the prompt-control check. |
| Width and height of generated images | They were not judged. |

### Model Card Claims We Did Not Check

- The adapters make generation up to 6.3x faster than the base model.
- Only the adapter's own step count and sigma schedule are supported, not other steps, schedules, or CFG values.
- The 6-step adapter does not guarantee outputs identical to the base model.
- Higher resolutions such as 2K are outside the training coverage and quality may vary.
- The adapters were trained at 1K resolution only, with editing of up to 3 reference images.
- Text-to-image output starts at a width and height of 1024 pixels.
- The adapter is not a standalone model and needs the Qwen-Image-2.1 base weights.
- Text-to-image and image editing run with the same pipeline and need a CUDA GPU. This service's NF4 workflow also ran on the MacBook M1 16GB.

## License

| Covers | License | Commercial use |
| :---: | --- | --- |
| This service | [`LICENSE`](LICENSE) | ✓ |
| Qwen/Qwen-Image-2.1 weights | [`MODEL_LICENSE`](https://huggingface.co/Qwen/Qwen-Image-2.1/blob/main/LICENSE) | ✗ |
| PrunaAI/P-Q-Image-2.1 LoRA | [`MODEL_LICENSE`](https://huggingface.co/PrunaAI/P-Q-Image-2.1/blob/main/LICENSE) | ✗ |
