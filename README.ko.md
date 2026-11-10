[English](README.md) · [한국어](README.ko.md) · [中文](README.zh.md)

# image-to-video-wan

이미지 한 장과 설명 글을 받아 Wan-AI/Wan2.2-TI2V-5B로 짧은 영상을 만들어 돌려주는 서비스입니다.

![수족관 창 앞에 사람이 서 있는 이미지와 설명 글을 넣으면 해파리가 떠다니는 영상이 나오는 화면](docs/images/demo.gif)

## 빠른 시작

[model-compose](https://github.com/hanyeol/model-compose) 0.4.120 이상이 필요합니다.

[uv](https://docs.astral.sh/uv/)로 설치:

```bash
uv pip install model-compose
```

또는 pip로 설치:

```bash
pip install model-compose
```

이 저장소 클론:

```bash
git clone https://github.com/MindrLabs/image-to-video-wan
cd image-to-video-wan
```

실행:

```bash
model-compose up
```

API 주소는 http://localhost:8080/api, Gradio UI 주소는 http://localhost:8081 입니다.

## 요약

RTX 4090은 5.04초 영상 하나를 만드는 데 중앙값 526초 걸려 소비자용 GPU 한 장에 대한 모델 카드의 속도 주장을 만족했습니다.

| 기기 | 상태 | RTF | 최대 메모리 (MiB) |
| :---: | --- | --- | --- |
| DGX Spark | 측정 | 281 | 30013 (VRAM) |
| RTX 4090 | 측정 | 104 | 23942 (VRAM) |
| MacBook M1 16GB | 미측정 | — | — |

## 측정 환경

model-compose 0.4.120 버전으로 측정했습니다. 통계는 95% 구간을 쓰고 1.05배 미만 속도 차이는 판정하지 않습니다.

| 입력 | 가중치 | 주장 판정 규칙 |
| :---: | --- | --- |
| 입력 3개 × 반복 1회 | [Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/tree/921dbaf3f1674a56f47e83fb80a34bac8a8f203e) | fps = 24; pixels = 720p; aspect_ratio_error ≤ 0.05 |

## 결과

### 이미지로 영상 만들기

이미지 한 장을 영상의 시작으로 삼고 움직임을 설명하는 글을 함께 넣어 영상을 만듭니다.

| 주장 | 확인 방법 | 기기 | 통과 |
| :---: | --- | --- | --- |
| TI2V-5B는 24fps 영상을 생성합니다. | 규칙 판정 | DGX Spark | ✓ |
| TI2V-5B는 24fps 영상을 생성합니다. | 규칙 판정 | RTX 4090 | ✓ |
| TI2V-5B는 720P 영상을 생성합니다. | 규칙 판정 | DGX Spark | ✓ |
| TI2V-5B는 720P 영상을 생성합니다. | 규칙 판정 | RTX 4090 | ✓ |
| TI2V-5B 영상은 입력 이미지의 가로세로 비율을 유지합니다. | 규칙 판정 | DGX Spark | ✓ |
| TI2V-5B 영상은 입력 이미지의 가로세로 비율을 유지합니다. | 규칙 판정 | RTX 4090 | ✓ |
| TI2V-5B는 이미지를 따라가는 영상을 만듭니다. | 사람 판정 | — | ✓ |
| TI2V-5B는 소비자용 GPU 한 장에서 5초 길이 720P 영상을 9분 안에 생성합니다. | 사람 판정 | — | ✓ |

DGX Spark, RTX 4090 기기에서 측정했습니다. MacBook M1 16GB 기기에서는 측정하지 못했습니다. RTF는 처리에 걸린 시간을 오디오나 영상 길이로 나눈 값이라 낮을수록 빠릅니다. 각 칸은 측정한 호출의 중앙값이며 다른 GPU 작업 중에 한 호출은 빼고 계산합니다.

| 기기 | RTF | 호출당 초 | 최대 메모리 (MiB) | aspect_ratio_error | duration_sec | fps | height | pixels | width |
| :---: | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DGX Spark | 281 | 1419 | 30013 (VRAM) | 0.0256 | 5.04 | 24 | 896 | 882688 | 1013 |
| RTX 4090 | 104 | 526 | 23942 (VRAM) | 0.0256 | 5.04 | 24 | 896 | 882688 | 1013 |

![이미지로 영상 만들기](docs/images/image-to-video-aquarium-jellyfish.gif)

> A girl in a school uniform stands still before a giant aquarium window, looking up as a huge glowing jellyfish drifts slowly with its long tentacles trailing, small fish swim past, sunlight ripples down through the water, static camera.

![이미지로 영상 만들기](docs/images/image-to-video-falling-crosswalk.gif)

> A girl in a school uniform drifts down through the air toward a sunlit crosswalk, her short hair and pleated skirt flutter in the wind, she reaches out with both arms, rippling light shimmers across the asphalt below and her shadow grows on the road, the camera slowly arcs a little around her, shifting the viewing angle.

![이미지로 영상 만들기](docs/images/image-to-video-train-turtle.gif)

> A girl sits on a train bench gazing out the window as a giant turtle glides past over a ruined, overgrown city, its eye blinking slowly, warm sunbeams shift across the carriage as the train sways gently, static camera.

## 추천

- RTX 4090에서는 GPU 메모리를 최대 23942 MiB 썼으니 실행 전에 그만큼 비워 두세요.

## 한계와 측정하지 않은 것

- 입력 3개를 한 번씩만 생성했으니 사람이 확인한 움직임 판정은 이 출력들에만 해당합니다.

### 측정 조건

| 항목 | 내용 |
| :---: | --- |
| 제약 | 측정 때 서버 환경에 PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True를 설정했습니다. 이 설정이 없으면 RTX 4090에서 VAE 디코딩 중 메모리가 부족했습니다. |
| 제약 | 측정은 WAN_CPU_OFFLOAD=true로 돌렸고 model-compose.yml의 기본값은 false입니다. |
| 주장 판정 규칙 | 규칙이 있는 주장은 기기마다 출처가 밝힌 숫자와 대조해 판정했습니다. |

### 측정하지 못한 것

| 대상 | 항목 | 이유 |
| :---: | --- | --- |
| MacBook M1 16GB | 모든 측정 | 모델의 flash-attn 의존성을 Apple Silicon에 설치할 수 없었습니다. |
| DGX Spark / RTX 4090 | RTF | 다른 GPU 작업 없이 두 기기에서 잰 입력이 3개뿐이고 최소 20개가 필요합니다. |
| DGX Spark / RTX 4090 | aspect_ratio_error | 두 기기에서 채점한 입력이 3개뿐이고 최소 20개가 필요합니다. |

### 확인하지 않은 모델 카드 주장

- 단일 GPU용 TI2V-5B 명령은 RTX 4090처럼 VRAM이 24GB인 GPU에서 돌아갑니다.

## 라이선스

| 대상 | 라이선스 | 상업적 이용 |
| :---: | --- | --- |
| 이 서비스 | [`LICENSE`](LICENSE) | ✓ |
| Wan2.2-TI2V-5B 가중치 | [`MODEL_LICENSE`](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B/blob/main/README.md) | ✓ |
