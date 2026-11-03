[English](README.md) · [한국어](README.ko.md) · [中文](README.zh.md)

# image-to-video-wan

本服务接收一张图片和描述动作的提示词，用 Wan-AI/Wan2.2-TI2V-5B 生成一段从这张图片开始的短视频。

测量设备：DGX Spark、RTX 4090；未与其他模型比较。
标为“人工判定”的结果是人工在看到代理建议之前作出的判定。
每条功能观察都由人工直接看或听输出后写下，代理草稿在此之后才显示。

![输入水族馆图片和提示词后得到水母漂浮视频的网页界面。](docs/images/demo.gif)

## 模型

Wan2.2 模型卡称 TI2V-5B 可从文本和图片生成 720P、24fps 视频，并能在 RTX 4090 这样的单张消费级 GPU 上运行。
模型卡还称输出比例跟随输入图片，并给出了在单张消费级 GPU 上生成一段短 720P 视频的时间上限。

权重：[Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/tree/921dbaf3f1674a56f47e83fb80a34bac8a8f203e)

## 快速开始

需要 [model-compose](https://github.com/hanyeol/model-compose) 0.4.120 或更高版本。

使用 [uv](https://docs.astral.sh/uv/) 安装：

```bash
uv pip install model-compose
```

或使用 pip 安装：

```bash
pip install model-compose
```

克隆此仓库：

```bash
git clone https://github.com/MindrLabs/image-to-video-wan
cd image-to-video-wan
```

运行：

```bash
model-compose up
```

API 地址为 http://localhost:8080/api 。

Gradio 界面地址为 http://localhost:8081 。

## 摘要

1. 在 RTX 4090 和 DGX Spark 上测量，两台机器都无失败地生成了 6 段视频。
2. 两台机器的输出都通过了 720P 面积、24fps 和输入图片比例规则，人工判定也通过了 RTX 4090 上的速度声明以及视频跟随图片这两项。
3. 三张图片都在固定镜头下出现了提示词描述的动作，如水母、水波和窗外风景，但有两段视频里的人物面部变形。
4. 只有 3 张图片，无法从统计上确认 RTX 4090 和 DGX Spark 的速度差异。

| 设备 | 状态 | RTF | 峰值内存 (MiB) |
| :---: | --- | --- | --- |
| DGX Spark | 已测量 | 276 | 30135 (VRAM) |
| RTX 4090 | 已测量 | 102 | 24050 (VRAM) |

## 测量环境

功能输出通过同一测量流程按每个输入和设置各生成一次，每个输出的观察都由人工确认。

| 项目 | 值 |
| :---: | --- |
| 输入 | 3 个输入 × 2 次重复 |
| 检查点 | [Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/tree/921dbaf3f1674a56f47e83fb80a34bac8a8f203e) |
| model-compose | 0.4.118 |
| 声明判定规则 | fps = 24; pixels = 720p; aspect_ratio_error ≤ 0.05 |
| 统计 | 95% 置信区间，低于 1.05× 的速度差异不作判定 |

## 我的设备能运行吗？

- 速度用 RTF 表示，即生成时间除以输出视频时长，越低越快。
- RTF 中位数在 RTX 4090 上为 102，在 DGX Spark 上为 276。
- 一段 5.04 秒视频在 RTX 4090 上用时 514 到 518 秒，在 DGX Spark 上用时 1338 到 1420 秒。
- GPU 显存峰值在 RTX 4090 上为 24050 MiB，在 DGX Spark 上为 30135 MiB。

### 设备比较

| 设备 | 指标 | 输入 | 估计 | 区间 | 结果 |
| :---: | --- | --- | --- | --- | --- |
| DGX Spark vs RTX 4090 | RTF | 3 | — | — | 无法判定 |
| DGX Spark vs RTX 4090 | aspect_ratio_error | 3 | — | — | 无法判定 |

## 各功能结果

| 问题 | 结果 |
| :---: | --- |
| 一张图片加一句简短提示词，能把这个场景变成动起来的视频吗？ | 可以，三段视频都从各自的图片开始并按提示词动了起来，但其中两段的人物面部变形。 |

### 单图生成视频（5B）

输入三张图片，每张配一段描述场景动作和固定镜头的英文提示词。

| 声明 | 核对方式 | 设备 | 结果 |
| :---: | --- | --- | --- |
| TI2V-5B 生成 24fps 视频。 | 规则判定 | DGX Spark | 通过 |
| TI2V-5B 生成 24fps 视频。 | 规则判定 | RTX 4090 | 通过 |
| TI2V-5B 生成 720P 视频。 | 规则判定 | DGX Spark | 通过 |
| TI2V-5B 生成 720P 视频。 | 规则判定 | RTX 4090 | 通过 |
| TI2V-5B 视频保持输入图片的比例。 | 规则判定 | DGX Spark | 通过 |
| TI2V-5B 视频保持输入图片的比例。 | 规则判定 | RTX 4090 | 通过 |
| TI2V-5B 生成跟随图片的视频。 | 人工判定 | — | 通过 |
| TI2V-5B 在单张消费级 GPU 上 9 分钟内生成一段 5 秒 720P 视频。 | 人工判定 | — | 通过 |

| 设备 | RTF | 峰值内存 (MiB) | aspect_ratio_error | duration_sec | fps | height | pixels | width | 声明 |
| :---: | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DGX Spark | 276 | 30135 (VRAM) | 0.0177 | 5.04 | 24 | 1003 | 878592 | 928 | 通过 |
| RTX 4090 | 102 | 24050 (VRAM) | 0.0177 | 5.04 | 24 | 1003 | 878592 | 928 | 通过 |

每格为所测调用的中位数，计算时不计入在其他 GPU 任务期间进行的调用。

| 设备 | 输入 | 秒 | RTF | 峰值内存 (MiB) |
| :---: | --- | --- | --- | --- |
| RTX 4090 | aquarium-jellyfish | 518 | 103 | 24008 (VRAM) |
| RTX 4090 | flooded-city | 514 | 102 | 24050 (VRAM) |
| RTX 4090 | train-turtle | 518 | 103 | 23928 (VRAM) |

![单图生成视频（5B）、RTX 4090](docs/images/shared/animate-5b-v6.nvidia-rtx-4090-24gb.frames.jpg)

RTX 4090：取自水族馆、被淹城市以及火车与海龟视频的画面。

行（从上到下）：

- aquarium-jellyfish: prompt=A girl in a school uniform stands still before a giant aquarium window, looking up as a huge glowing jellyfish drifts slowly with its long tentacles trailing, small fish swim past, sunlight ripples down through the water, static camera.; image
- flooded-city: prompt=A girl stands in shallow clear water on a flooded crosswalk between towering scaffolded buildings, looking up at a golden sky, clouds drift slowly, sheets of paper flutter in the wind, gentle ripples spread across the water, static camera.; image
- train-turtle: prompt=A girl sits on a train bench gazing out the window as a giant turtle glides past over a ruined, overgrown city, its eye blinking slowly, warm sunbeams shift across the carriage as the train sways gently, static camera.; image

- 水族馆视频中水母在动，小鱼在游，镜头保持固定。
- 被淹城市视频中水面泛起波纹，旗帜飘动，镜头保持固定。
- 火车与海龟视频中海龟眨眼，窗外风景掠过，光线散射，镜头保持固定。

## 建议

- RTX 4090 几乎用满了显存，建议不要在同一块 GPU 上同时运行其他任务。
- 人物面部可能变形，建议看完整段视频再使用。

## 局限与未测量的内容

### 缺陷

- 被淹城市的视频中人物面部变形。
- 火车与海龟的视频中光线过强，人物面部变形。

### 其他局限

- DGX Spark 与 RTX 4090 未比较 RTF：两台机器在没有其他 GPU 任务的情况下只测了 3 个输入，至少需要 20 个。
- DGX Spark 与 RTX 4090 未比较 aspect_ratio_error：两台机器只评分了 3 个输入，至少需要 20 个。
- 此处未测试：TI2V-5B 单 GPU 命令可在 RTX 4090 这样的 24GB GPU 上运行。
- 服务器设置了 PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True，因为不设置时 RTX 4090 在 VAE 解码时显存不足。
- 速度和人工判定只用了 3 张图片。
- 模型卡写的单 GPU 显存要求只在一台 RTX 4090 上看过，该声明本身没有单独测试。
- 带规则的声明已在每台设备上与来源给出的数字对照判定。

## 许可证

| 适用对象 | 许可证 | 商业使用 |
| :---: | --- | --- |
| 本服务 | [`LICENSE`](LICENSE) | ✓ |
| Wan2.2-TI2V-5B 权重 | [`MODEL_LICENSE`](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/blob/main/README.md#license-agreement) | ✓ |
