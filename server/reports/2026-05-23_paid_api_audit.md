# Zealman v8.51 付费 API 风险审计

审计时间: 2026-05-23

目的: 确认当前 Zealman ComfyUI v8.51 实例是否已经接入 AutoDL 平台付费模型 API, 以及运行视频/图像工作流时是否可能自动扣除 AutoDL API 费用。

说明: 本报告不记录 SSH 密码、API key、token 原文或私人令牌。

## 结论

当前没有发现 Zealman 主面板或 ComfyUI 视频生成链路已经接入有效的 AutoDL 付费模型 API。

当前明确会产生费用的是:

- AutoDL 实例运行费用, 即 GPU 服务器按时计费。
- AutoDL 日常费用/实例保留费用, 以平台账单为准。

当前没有证据表明普通 ComfyUI 本地工作流生成视频时会自动调用 AutoDL 付费模型 API 扣费。

## 已检查内容

- 运行进程:
  - Zealman 面板 `node server.js`
  - ComfyUI `python main.py --port 6006`
  - JupyterLab
  - TensorBoard
  - AutoPanel
- Zealman 主配置:
  - `/root/zealman-app/config.json`
  - `/root/zealman-app/modelgo.yaml`
  - `/root/zealman-app/modelgo1.yaml`
- Zealman 路由代码:
  - `/root/zealman-app/routes`
  - `/root/zealman-app/server.js`
- Wuli 画布服务:
  - `/root/zealman-app/Wuli-API`
- ComfyUI 工作流:
  - `/root/ComfyUI/user/default/workflows`
- ComfyUI 自定义节点:
  - `/root/ComfyUI/custom_nodes`

## 环境变量审计

系统环境和运行中的 Zealman/ComfyUI 进程里没有发现以下有效密钥类变量:

- OpenAI API key
- MiniMax API key
- DashScope API key
- BizyAir API key
- ModelScope API key
- AutoDL 付费模型 API key

运行进程中存在 AutoDL 平台服务变量, 例如服务公网地址、区域、容器 ID、AutoPanel token 等。这些是 AutoDL 为实例和面板服务注入的运行环境信息, 用于访问 WebUI/AutoPanel, 不等同于可调用付费模型的 API key。

## Wuli 画布 API 情况

Wuli 画布代码里确实支持以下外部 API:

- Comfly / OpenAI 兼容接口
- ModelScope
- AutoDL OpenAI/Gemini 兼容接口

但当前状态:

- Wuli 画布端口 `6010` 没有运行。
- `/root/zealman-app/Wuli-API/API/.env` 中:
  - `COMFLY_API_KEY`: 3 位占位符
  - `MODELSCOPE_API_KEY`: 3 位占位符
  - `AUTODL_API_KEY`: 3 位占位符
- 这些不像有效密钥。

结论: Wuli 画布具备接入外部 API 的能力, 但当前没有配置有效 AutoDL 付费模型 API key。除非在画布设置里主动填写有效 key 并调用相关接口, 否则不会自动扣外部 API 费用。

## 工作流关键词说明

部分工作流 JSON 里出现 `token`、`gpt` 等关键词, 这不一定代表在线 API 调用:

- `token` 很多时候是文本编码、tokenizer、CLIP/Qwen/Gemma token 相关字段。
- `gpt` 可能出现在提示词、节点名、备注或本地视觉语言/提示词优化链路中。
- 需要根据节点类型和实际请求行为判断, 不能只凭关键词判定会扣费。

第一批工作流中未发现明确要求 AutoDL 付费 API key 才能运行的迹象。优先验证的 Wan2.2、LTX2.3、Z-Image、Qwen/Flux 等链路主要使用本地/公共模型区挂载模型。

## 当前风险点

1. Wuli 画布页面可能允许用户填写 AutoDL/ModelScope/Comfly API key。
2. 某些自定义节点具备在线 API 能力, 但需要配置 key 或手动选择对应节点。
3. Zealman 面板的“API生成”是把本地 ComfyUI 工作流封装成可调用接口, 不等同于接入 AutoDL 付费模型 API。
4. 如果后续导入第三方工作流, 需要检查是否包含 OpenAI、MiniMax、DashScope、ModelScope、AutoDL API 节点。

## 项目规则

- 不使用 AutoDL 平台付费模型 API, 除非用户明确要求并确认费用。
- 不在 Zealman/Wuli/ComfyUI 里填写 `AUTODL_API_KEY`, 除非用户明确批准。
- 用户提供的闭源辅助 API, 如 GPT image2、MiniMax TTS、LLM、字幕、翻译等, 应使用用户指定的平台和密钥, 不默认使用 AutoDL 平台 API。
- 每次导入新工作流前, 先检查是否有在线 API 节点或密钥字段。
- 批量生成前, 先做一次小样例并确认日志里没有外部 API 调用。

## 后续建议

1. 第一批验证只运行本地模型工作流。
2. 不点击或配置 Wuli 画布里的 AutoDL API key 相关设置。
3. 若需要外部 API, 单独建立 `.env` 模板, 明确供应商、费用和用途。
4. API 集成统一走我们项目的配置规范, 不混用 AutoDL 付费模型 API。
