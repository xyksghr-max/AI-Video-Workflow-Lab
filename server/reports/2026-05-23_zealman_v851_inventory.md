# Zealman ComfyUI v8.51 服务器体检报告

体检时间: 2026-05-23 00:04 CST

说明: 本报告只记录服务器环境和路径信息, 不记录 SSH 密码、API key 或私人令牌。

## 实例概况

- 平台: AutoDL / SeetaCloud 连接域
- 主机名: `autodl-pro-7794a4dabdbf`
- 系统: Ubuntu, Linux kernel `5.15.0-78-generic`
- 当前用户: `root`
- AI 应用: Zealman ComfyUI v8.51
- ComfyUI 根目录: `/root/ComfyUI`
- Zealman 面板目录: `/root/zealman-app`
- 工作流目录: `/root/ComfyUI/user/default/workflows`
- 快捷/面板工作流目录: `/root/zealman-app/workflows`

## GPU 和运行环境

- GPU: NVIDIA GeForce RTX 4090
- 显存: 49,140 MiB
- 体检时显存占用: 约 465 MiB
- Driver: `580.105.08`
- `nvidia-smi` CUDA: `13.0`
- Python: `3.12.3`
- PyTorch: `2.8.0+cu128`
- Torch CUDA: `12.8`
- ComfyUI API 报告版本: `0.20.1`

## 磁盘情况

- 系统盘 `/`: 30G, 已用 15G, 剩余 16G
- `/root/ComfyUI`, `/root/ComfyUI/output`, `/root/autodl-tmp` 都在 30G overlay 系统盘上
- 公共模型区: `/.autodl-model/data`
- 公共模型区容量: 20T, 已用 14T, 剩余 6.2T, 只读挂载
- `/root/ComfyUI`: 约 3.9G
- `/root/ComfyUI/input`: 约 320M
- `/root/ComfyUI/output`: 约 256M
- `/root/ComfyUI/temp`: 约 4K
- `/root/autodl-tmp`: 约 2.2M

结论: 当前实例没有单独的大容量私有数据盘。Zealman 内置大模型主要通过符号链接指向 AutoDL 公共模型区, 不占系统盘。后续如果要下载私有大模型、保存大量输出或缓存, 必须先扩容或确认可用持久化数据盘, 不能直接放 `/root`。

## 端口和服务

- `6006`: ComfyUI, 监听 `127.0.0.1:6006`
- `6008`: Zealman 面板, Node 服务
- `8888`: JupyterLab
- `6007`: TensorBoard
- `2022`: AutoPanel
- `6010`: Wuli 画布服务, 默认按需启动, 当前不是开机自启

ComfyUI 启动命令:

```bash
/root/miniconda3/bin/python main.py --port 6006 --listen 127.0.0.1 --enable-cors-header "*"
```

ComfyUI 启动脚本:

```text
/root/zealman-app/start-comfyui.sh
```

该文件是符号链接, 指向 AutoDL 用户区的 Zealman 启动脚本。脚本会启动前更新模型符号链接, 然后启动 ComfyUI。

Zealman 面板启动脚本:

```text
/root/zealman-app/start-services.sh
```

面板配置中 `autoStartComfyUI` 为 `true`, `enableSymlink` 为 `true`。

## 工作流概况

ComfyUI 默认工作流目录:

```text
/root/ComfyUI/user/default/workflows
```

统计结果:

- 工作流 JSON: 193 个
- 分类目录: 17 个
- 自定义节点目录: 129 个

分类数量:

- A图像-Qwen生成: 11
- B图像-Qwen编辑: 16
- C图像-Zimage: 17
- D图像-Flux: 19
- E图像-SDXL-IL: 3
- F图像-FireRed图像编辑: 2
- G视频-Wan图生: 25
- H视频-LTX: 16
- J视频-对口型数字人: 13
- K视频-换装影视: 6
- L视角-高斯泼渐: 2
- M高清放大去水印: 9
- N声音克隆: 6
- P视频-动作迁移: 13
- Y通用制作: 8
- Z工具设置: 9
- OLD: 18

## 模型概况

模型主目录:

```text
/root/ComfyUI/models
```

重要结论:

- 本地模型目录直接占用约 597M。
- 模型符号链接约 999 个。
- 大模型实际位于 `/.autodl-model/data` 或 `/.autodl/...`。
- 当前无需立刻下载 Wan2.2 或 LTX2.3 主模型。

已确认存在的关键模型能力:

- Wan2.2 I2V A14B FP8: 已有 high/low noise FP8 scaled 版本。
- Wan2.2 T2V A14B FP8: 已有 high/low noise FP8 scaled 版本。
- Wan2.2 Animate 14B FP8: 已有 KJ / v2 相关版本。
- Wan2.2 Fun Control FP8: 已有 high/low noise 14B FP8 scaled 版本。
- Wan2.2 Fun Inpaint FP8: 已有 high/low noise 14B FP8 scaled 版本。
- Wan 首尾帧/FMLF 类: 工作流已存在; 精确模型依赖以对应工作流为准。
- LTX-2.3 22B dev FP8: 已有 `ltx-2.3-22b-dev-fp8.safetensors`。
- Gemma 3 12B FP4 mixed: 已有 `gemma_3_12B_it_fp4_mixed.safetensors`。
- LTX-2.3 spatial upscaler x2: 已有 `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`。
- SeedVR2 / FlashVSR / LTX 修复放大相关模型: 已有多种版本。
- InfiniteTalk / LTX 数字人相关模型: 已有单人、多人和 LTX 音画同步相关模型。

## 当前风险和规则

- 30G 系统盘只剩约 16G, 不能直接下载新大模型。
- 输出视频、上传素材和临时缓存需要控制体积。
- 大文件下载必须先确认目标目录不是系统盘, 并使用 `screen`。
- 公共模型区是只读模型仓库, 适合复用内置模型, 不适合写入我们的私有文件。
- 任何“去水印”能力只用于自有或已授权素材的修复处理, 不用于规避第三方版权标识。

## 下一步建议

1. 先从内置模型和内置工作流开始跑小样例。
2. 第一轮只用低分辨率、短帧数、低步数验证链路。
3. 优先验证角色图、分镜图、Wan2.2 图生视频、首尾帧、LTX2.3、口型和动作迁移。
4. 确认输出路径和单次任务体积后, 再决定是否扩容系统盘或挂载数据盘。
