[English](README.md) · [한국어](README.ko.md) · [中文](README.zh.md)

# image-to-video-wan

This service takes one picture and a prompt that describes the motion, and returns a short video clip that starts from that picture, made with Wan-AI/Wan2.2-TI2V-5B.

Measured on DGX Spark, RTX 4090; it is not compared with other models.
Results marked “Judged by a person” are a person's verdicts, given before the agent's proposal was shown.
A person wrote every feature observation from the output itself; the agent's draft was shown only afterwards.

## Model

The Wan2.2 model card says TI2V-5B makes 720P, 24fps video from text and images and runs on a single consumer GPU such as the RTX 4090.
The card also says the output follows the aspect ratio of the input picture and gives a time limit for a short 720P clip on a single consumer GPU.

Weights: [Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/tree/921dbaf3f1674a56f47e83fb80a34bac8a8f203e)

## Demo

![A web page where an aquarium picture and a prompt go in and a clip of a drifting jellyfish comes out.](docs/images/demo.gif)

## Quick Start

It needs [model-compose](https://github.com/hanyeol/model-compose) 0.4.120 or later.

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
git clone https://github.com/MindrLabs/image-to-video-wan
cd image-to-video-wan
```

Run it:

```bash
model-compose up
```

The API listens on http://localhost:8080/api.

The Gradio UI opens at http://localhost:8081.

## Summary

1. It ran on the RTX 4090 and the DGX Spark, and both made all 6 clips without a failure.
2. On both machines the clips passed the 720P area, 24fps and input aspect ratio rules, and a person passed the speed claim on the RTX 4090 and the claim that the clip follows the picture.
3. All three pictures moved the way the prompt asked under a fixed camera, with jellyfish, ripples and passing scenery, but a face came out distorted in two clips.
4. With only 3 pictures, the speed difference between the RTX 4090 and the DGX Spark could not be confirmed statistically.

| Machine | Status | RTF | Peak memory (MiB) |
| :---: | --- | --- | --- |
| DGX Spark | Measured | 276 | 30135 (VRAM) |
| RTX 4090 | Measured | 102 | 24050 (VRAM) |

## Setup

Feature outputs were made once per input and setting through the same measurement path, and a person confirmed an observation of every output.

| Item | Value |
| :---: | --- |
| Inputs | 3 inputs × 2 repeats |
| Checkpoint | [Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/tree/921dbaf3f1674a56f47e83fb80a34bac8a8f203e) |
| model-compose | 0.4.118 |
| Claim rules | fps = 24; pixels = 720p; aspect_ratio_error ≤ 0.05 |
| Statistics | 95% intervals, speed differences under 1.05× are not called |

## Does It Run on My Device?

- Speed is given as RTF, the generation time divided by the length of the output clip, so lower is faster.
- The median RTF was 102 on the RTX 4090 and 276 on the DGX Spark.
- One 5.04-second clip took 514 to 518 seconds on the RTX 4090 and 1338 to 1420 seconds on the DGX Spark.
- GPU memory peaked at 24050 MiB on the RTX 4090 and 30135 MiB on the DGX Spark.

### Machine Comparisons

| Machines | Metric | Inputs | Estimate | Interval | Result |
| :---: | --- | --- | --- | --- | --- |
| DGX Spark vs RTX 4090 | RTF | 3 | — | — | Not judged |
| DGX Spark vs RTX 4090 | aspect_ratio_error | 3 | — | — | Not judged |

## Results by Feature

| Question | Result |
| :---: | --- |
| Can one picture and a short prompt turn that scene into a moving clip? | Yes, all three clips started from their picture and moved the way the prompt asked, but a face came out distorted in two of them. |

### Animate a Still (5B)

Three pictures went in, each with an English prompt that describes the motion in the scene and a static camera.

| Claim | Check | Machine | Result |
| :---: | --- | --- | --- |
| TI2V-5B generates 24fps video. | Checked by rule | DGX Spark | Pass |
| TI2V-5B generates 24fps video. | Checked by rule | RTX 4090 | Pass |
| TI2V-5B generates 720P video. | Checked by rule | DGX Spark | Pass |
| TI2V-5B generates 720P video. | Checked by rule | RTX 4090 | Pass |
| The TI2V-5B video keeps the aspect ratio of the input image. | Checked by rule | DGX Spark | Pass |
| The TI2V-5B video keeps the aspect ratio of the input image. | Checked by rule | RTX 4090 | Pass |
| TI2V-5B turns an image into a video that follows the image. | Judged by a person | — | Pass |
| TI2V-5B generates a 5-second 720P video in under 9 minutes on a single consumer-grade GPU. | Judged by a person | — | Pass |

| Machine | RTF | Peak memory (MiB) | aspect_ratio_error | duration_sec | fps | height | pixels | width | Claims |
| :---: | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DGX Spark | 276 | 30135 (VRAM) | 0.0177 | 5.04 | 24 | 1003 | 878592 | 928 | Pass |
| RTX 4090 | 102 | 24050 (VRAM) | 0.0177 | 5.04 | 24 | 1003 | 878592 | 928 | Pass |

Each cell is the median over measured calls, leaving out any call made during other GPU work.

| Machine | Input | Seconds | RTF | Peak memory (MiB) |
| :---: | --- | --- | --- | --- |
| RTX 4090 | aquarium-jellyfish | 518 | 103 | 24008 (VRAM) |
| RTX 4090 | flooded-city | 514 | 102 | 24050 (VRAM) |
| RTX 4090 | train-turtle | 518 | 103 | 23928 (VRAM) |

![Animate a still (5B), RTX 4090](docs/images/shared/animate-5b-v6.nvidia-rtx-4090-24gb.frames.jpg)

RTX 4090: Frames from the aquarium, flooded city and train and turtle clips.

Rows, top to bottom:

- aquarium-jellyfish: prompt=A girl in a school uniform stands still before a giant aquarium window, looking up as a huge glowing jellyfish drifts slowly with its long tentacles trailing, small fish swim past, sunlight ripples down through the water, static camera.; image
- flooded-city: prompt=A girl stands in shallow clear water on a flooded crosswalk between towering scaffolded buildings, looking up at a golden sky, clouds drift slowly, sheets of paper flutter in the wind, gentle ripples spread across the water, static camera.; image
- train-turtle: prompt=A girl sits on a train bench gazing out the window as a giant turtle glides past over a ruined, overgrown city, its eye blinking slowly, warm sunbeams shift across the carriage as the train sways gently, static camera.; image

- In the aquarium clip the jellyfish moved, small fish swam and the camera stayed fixed.
- In the flooded city clip the water rippled, a flag waved and the camera stayed fixed.
- In the train and turtle clip the turtle blinked, the scenery passed outside the window, the light scattered and the camera stayed fixed.

## Recommendations

- The RTX 4090 uses nearly all of its GPU memory, so do not run other work on the same GPU.
- Faces can come out distorted, so watch the whole clip before using it.

## Limitations and What We Did Not Measure

### Defects

- In the flooded city clip the girl's face came out distorted.
- In the train and turtle clip the light grew too strong and the girl's face came out distorted.

### Other Limitations

- DGX Spark and RTX 4090 were not compared on RTF: Only 3 inputs were measured on both machines without other GPU work, and at least 20 are needed.
- DGX Spark and RTX 4090 were not compared on aspect_ratio_error: Only 3 inputs were scored on both machines, and at least 20 are needed.
- Not tested here: The single-GPU TI2V-5B command runs on a GPU with 24GB of VRAM such as the RTX 4090.
- The server sets PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True, because without it the VAE decode ran out of memory on the RTX 4090.
- Speed and the person's verdicts come from only 3 pictures.
- The card's memory requirement for a single GPU was seen on one RTX 4090 only, and that claim itself was not tested.
- Claims with a rule were checked on each machine against the number their source gives.

## License

| Covers | License | Commercial use |
| :---: | --- | --- |
| This service | [`LICENSE`](LICENSE) | ✓ |
| Wan2.2-TI2V-5B weights | [`MODEL_LICENSE`](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/blob/main/README.md#license-agreement) | ✓ |
