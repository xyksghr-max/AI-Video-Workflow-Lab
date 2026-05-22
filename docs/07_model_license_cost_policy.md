# 模型费用和授权边界

## 核心结论

Zealman ComfyUI v8.51 里的大部分内置工作流是本地 ComfyUI 调用模型权重运行。正常使用这些本地/公共模型区挂载的模型时, 通常不会产生单次模型 API 调用费, 主要成本是 AutoDL GPU 服务器运行费、存储费和网络/实例相关费用。

但“能本地免费运行”不等于“所有模型都是开源且可随意商用”。模型要分为:

- 真正宽松开源或开放许可, 例如 Apache-2.0。
- 开放权重但带社区许可或商业限制。
- 官方模型的 ComfyUI 重打包、量化、FP8/GGUF 适配版。
- 社区 LoRA、蒸馏、混合版、加速版。
- 外部 API 模型, 按供应商 API 计费。

## 费用判断

### 通常只付服务器费

满足以下条件时, 一般只付服务器费:

- 工作流使用 `/root/ComfyUI/models` 中的本地模型或 AutoDL 公共模型区符号链接。
- 没有填写 OpenAI、MiniMax、DashScope、ModelScope、AutoDL API key。
- 工作流没有在线 API 节点。
- 任务在本机 GPU 上运行。

### 会产生额外 API 费用

满足以下任一条件时, 可能产生额外 API 费用:

- 使用 GPT image2 / OpenAI 图像 API。
- 使用 MiniMax TTS / 语音 API。
- 使用外部 LLM 做提示词扩写、翻译、字幕、分镜、角色设计。
- 在 Wuli 画布或 ComfyUI 节点里填写了外部 API key。
- 使用 AutoDL 平台模型 API key 调用 AutoDL 付费模型接口。

项目规则: 不默认使用 AutoDL 平台付费模型 API。除非用户明确批准, 不填写 `AUTODL_API_KEY`。

## 授权判断

### 可信但不全是“官方原版”

当前服务器上很多模型是:

- 官方模型的 ComfyUI 分包版本。
- 官方模型的 FP8 / scaled / GGUF 量化版本。
- 社区优化版、LoRA、工作流专用版。

这不等于残次品。很多 ComfyUI 工作流必须使用这些适配版本才能在单卡上跑。但它们也不等于原始官方权重。

### 典型授权例子

- Wan2.2 T2V/I2V 官方仓库标注 Apache-2.0。
- Alibaba-PAI Wan2.2 Fun Control / InP 标注 Apache-2.0。
- Lightricks LTX-2.3 使用 LTX-2 Community License Agreement, 不是 Apache-2.0。
- FLUX.1 dev 使用非商业许可, 不能简单当作可自由商用开源模型。
- Z-Image-Turbo 是 Tongyi-MAI 官方公开模型, 需按其模型页和许可证使用。

## 我们的使用策略

1. 测试阶段:
   - 优先使用 Zealman 内置工作流和已挂载模型。
   - 主要成本按服务器运行费计算。
   - 不接外部 API, 不填 AutoDL API key。

2. 生产阶段:
   - 每条工作流记录实际模型和许可证。
   - 商业发布前优先选择许可清晰的模型。
   - 对非商业许可、社区许可、灰色来源模型标红。
   - 二创和影视素材必须确保素材来源和使用权。

3. 外部 API 阶段:
   - 单独配置用户指定的 API key。
   - 单独记录供应商、用途、单价和调用量。
   - 不与 AutoDL 平台 API 混用。

## 验收规则

每条准备进入生产的工作流都要记录:

- 工作流路径
- 主模型
- LoRA / ControlNet / text encoder / VAE
- 是否本地运行
- 是否调用外部 API
- 模型许可证
- 是否适合商用
- 单次成本估算
- 输出样例和质量评价
