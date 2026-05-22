# Zealman v8.51 模型库存解释

体检时间: 2026-05-23

说明: 本文只解释当前服务器上看到的模型路径、符号链接和业务分类, 不记录 SSH 密码、API key 或私人令牌。

## 999 个符号链接是什么意思

`/root/ComfyUI/models` 里约有 999 个模型符号链接。它不等于 999 个独立大模型。

进一步统计:

- 模型符号链接: 999
- 跟随符号链接后可见的模型类文件路径: 1007
- 不同文件名: 864
- 不同符号链接目标: 890

产生差异的原因:

- 同一个模型可能同时出现在 `checkpoints` 和 `checkpoint`。
- 同一个 LTX 模型可能同时放在 `Ltx` 和 `Ltx-2.3` 目录。
- 一些工作流需要 VAE、text encoder、LoRA、ControlNet、检测器、分割器、upscaler, 这些都算模型文件, 但不是独立主模型。
- AutoDL 公共模型区使用 hash 路径或作者路径, 本地目录用符号链接映射成 ComfyUI 需要的名字。

结论: 这个镜像不是 999 个独立主模型, 而是几百个主模型、量化版本、LoRA、VAE、文本编码器和工具模型组成的模型资产库。

## 按目录统计

- `diffusion_models`: 329
- `loras`: 250
- `checkpoints`: 94
- `text_encoders`: 41
- `vae`: 30
- `ipadapter`: 25
- `embeddings`: 23
- `upscale_models`: 19
- `LLM`: 16
- `model_patches`: 14
- `sam2`: 12
- `ultralytics`: 12
- `insightface`: 11
- `mmaudio`: 9
- `stt`: 9

## 按领域粗略统计

以下数量按文件名或路径关键词粗略归类, 同一个模型可能被多个类别命中。

- Wan / 视频扩散: 164
- LTX / 视频扩散: 153
- Qwen 图像/编辑/VL/TTS: 134
- Flux / Kontext / Klein: 93
- 音频/语音/音乐: 84
- 检测/分割/姿态/人脸: 71
- Z-Image: 60
- 修复/放大/插帧: 52
- SDXL / Illustrious / 经典 SD: 47
- LLM / 提示词 / 视觉语言: 46
- FireRed: 9

## 官方性和可信度分层

不能只凭文件名说“100%官方原始权重”。当前应分为四层:

1. 官方原始或官方发布仓库:
   - Wan-AI 官方仓库存在 Wan2.2 I2V、T2V、Animate。
   - Lightricks 官方仓库存在 LTX-2.3 和 `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`。
   - Tongyi-MAI 官方仓库存在 Z-Image-Turbo。
   - Alibaba-PAI 官方仓库存在 Wan2.2-Fun-A14B-Control / InP。

2. 官方模型的 ComfyUI 重打包或量化版本:
   - 例如 Comfy-Org 的 Wan2.2 ComfyUI Repackaged FP8。
   - 例如 ComfyUI 需要的 split files、text encoder、VAE、FP8 scaled 版本。
   - 这类不是“残次品”, 而是为了单卡和 ComfyUI 节点运行做的格式适配或量化。

3. 社区优化、蒸馏、LoRA、混合版:
   - 例如 SmoothMix、Dasiwa、LightX2V、SVI、KJ scaled、Rapid AIO、各类镜头/动作/风格 LoRA。
   - 这类不能称为官方原版, 但可能更适合实际 ComfyUI 工作流和 4090-48G。

4. 未验收:
   - 只看到了文件或符号链接, 还没有在当前机器跑出样片。
   - 后续必须通过小样例验证显存、速度、画质和报错情况。

## 已核验的关键来源

- Wan-AI 官方 Wan2.2 I2V: https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B
- Wan-AI 官方 Wan2.2 T2V: https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B
- Wan-AI 官方 Wan2.2 Animate: https://huggingface.co/Wan-AI/Wan2.2-Animate-14B
- Comfy-Org Wan2.2 ComfyUI Repackaged: https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged
- Lightricks 官方 LTX-2.3: https://huggingface.co/Lightricks/LTX-2.3
- Tongyi-MAI 官方 Z-Image-Turbo: https://huggingface.co/Tongyi-MAI/Z-Image-Turbo
- Alibaba-PAI Wan2.2-Fun-A14B-Control: https://huggingface.co/alibaba-pai/Wan2.2-Fun-A14B-Control
- Alibaba-PAI Wan2.2-Fun-A14B-InP: https://huggingface.co/alibaba-pai/Wan2.2-Fun-A14B-InP

## 已看到的主要模型领域

### 视频生成

- Wan2.2 I2V / T2V / TI2V
- Wan2.2 Animate
- Wan2.2 Fun Control / Fun InP / Control Camera
- Wan 首尾帧、首中尾帧、长视频、StoryMem、SVI、LightX2V
- LTX-2 / LTX-2.3
- Hunyuan Video
- Mochi
- SkyReels / SteadyDancer / Ditto / Humo / Chrono 等相关模型或工作流依赖

### 图像生成和编辑

- Qwen Image / Qwen Image Edit / Qwen Layered
- Z-Image / Z-Image Turbo / Z-Image ControlNet / Z-Image LoRA
- Flux / Flux Kontext / Flux2 / Klein
- SDXL / Illustrious / NoobAI / Juggernaut / DreamShaper 等
- FireRed Image Edit
- HiDream
- ERNIE Image

### 语音、音频和音乐

- Qwen3 TTS
- FishAudio S2
- MMAudio
- ACE-Step 音乐/歌曲生成
- Whisper / wav2vec2
- MelRoFormer
- CLAP / vocoder / 音频 VAE

### 修复、放大和补帧

- SeedVR2
- FlashVSR
- SUPIR
- LTX spatial / temporal upscalers
- RIFE / GIMM-VFI
- ProPainter
- RTX 高清放大相关节点

### 控制、人脸、分割和姿态

- ControlNet / IPAdapter / InstantID / PuLID
- InsightFace
- YOLO / YOLO pose
- SAM2 / SAM3
- GroundingDINO
- RMBG / BiRefNet / VitMatte
- VitPose / SDPose / segformer clothes

### LLM、提示词和视觉语言

- Qwen3.6 GGUF
- Qwen3.5 / Qwen VL / Qwen 2.5 VL
- Gemma 3 text encoders
- Mistral text encoders for Flux2
- MiniCPM prompt generator
- Florence prompt generation
- ChatGLM

## 下一步验收方法

1. 先验证第一批工作流能否打开, 是否有缺失节点。
2. 用低分辨率、短帧数、小步数跑通样例。
3. 记录每条工作流实际加载的模型名、显存峰值、耗时和输出路径。
4. 对关键模型需要时再做 hash/source 级核验。
5. 只有确认缺失或效果不达标时, 才制定下载计划。
