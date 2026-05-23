# 字字动画 + Zealman API 集成说明

## 当前本地资料

本地已下载资料目录:

```text
D:\AI-Video-Workflow-Lab\字字工作流插件
```

目录内容:

- `可导入字字的API工作流/`
  - `A01-文生图-Qwen2512.json`
  - `G05-图生视频-Wan2.2+LightX2V最强动态.json`
  - `H08-文生视频-LTX2.0多模态全自动16秒低配.json`
  - `H27-LTX2.3首尾帧-自动写词版.json`
  - `H27-LTX2.3首尾帧.json`
  - `H28-LTX2.3-高动态图生视频.json`
  - `Y06-图像改为下一个场景图.json`
- `字字插件/image_plugins/AutoDL-zealman-ImageAPI/`
- `字字插件/video_plugins/AutoDL-zealman-VideoAPI/`
- `字字插件/image_plugins/flux2klein_multi_edit/`
  - 插件名: `flux2klein多图编辑`
  - 插件说明: OpenAI 风格 `/v1/images/edits` 多图编辑插件
  - 支持模型: `flux-2-klein-9b`, `flux-2-klein-4b`
- `字字插件/image_plugins/bizyair_2fen_image/`

该目录是第三方下载包, 当前只作为本地参考和安装源, 不提交 Git。

## 这套插件解决什么问题

有些 ComfyUI 工作流可以在 Zealman 镜像里正常运行, 但直接导入字字动画时可能出现节点识别不正确。这个插件的思路是:

1. 在 Zealman 面板里把已跑通的 ComfyUI 工作流转换成 API。
2. 字字动画不再直接解析复杂工作流 JSON。
3. 字字动画插件只调用 Zealman 的工作流 API, 并把文本、图片、种子、尺寸等参数映射到 `input_values`。

这样可以绕过字字动画对某些 ComfyUI 节点识别不完整的问题。

## 三种接入路线

字字动画接入当前 AI 工具生态现在分三条路线, 后续每条工作流先判断适合哪一种。

### 路线 A: 字字内置 ComfyUI 工作流模式

适合已经能被字字正确解析的 ComfyUI API 工作流。

核心流程:

1. 在 AutoDL 实例页点击 `WebUI-6006`, 打开真正的 ComfyUI 页面。
2. 在字字动画 `软件设置 -> 基础设置 -> 图片生成引擎设置` 中选择:
   - 引擎类型: `ComfyUI`
   - 服务器类型: `外部服务器`
   - 服务器地址: 填刚才打开的 ComfyUI 浏览器地址。
3. 点击测试, 需要通过 `/object_info` 和 WebSocket `/ws` 检查。
4. 在 ComfyUI 里打开目标工作流, 先用低参数跑通一次。
5. 从 ComfyUI 菜单导出 `API JSON`, 再导入字字动画的 `ComfyUI 工作流`。

注意:

- 这里填的是 ComfyUI 地址, 不是 Zealman 管理面板地址。
- 如果字字测试访问 `/object_info` 返回 `404 Not Found`, 通常说明填错了地址。
- 正确地址一般来自 AutoDL 的 `WebUI-6006`, 打开后浏览器地址可能表现为 HTTPS 映射域名和端口。
- 导入字字后如果看不到节点, 基本说明导出的不是 API JSON, 或导入了普通工作流 JSON。

### 路线 B: Zealman 自定义 API 插件模式

适合复杂工作流、字字解析不完整、节点太多或字段识别不稳定的工作流。

核心流程:

1. 在 ComfyUI 里打开目标工作流并跑通。
2. 从 ComfyUI 菜单导出 `API JSON`。
3. 打开 Zealman 管理面板 `WebUI-6008`。
4. 进入 `API生成`, 导入 API JSON。
5. 用智能筛选和人工检查确认要暴露的字段。
6. 在字字动画自定义插件里填写:
   - Base URL: Zealman 管理面板公网地址
   - Workflow ID: Zealman API 生成页给出的 `workflow_id`
   - 正向/负向提示词、图片、种子、尺寸、时长等字段映射

这条路线不依赖字字直接解析完整 ComfyUI 节点图, 更适合生产化接入。

### 路线 C: FLUX.2 Klein 专用图片编辑插件模式

适合快速多图图片编辑, 例如多参考图改图、角色图微调、分镜图局部重绘、画风统一等。

这条路线不是导入 ComfyUI 工作流, 也不是 Zealman 面板的 `/api/workflow/generate`。它调用的是一个 OpenAI 风格图片接口:

- 有参考图: `POST /v1/images/edits/jobs`
- 无参考图: `POST /v1/images/jobs`
- 轮询结果: `GET /v1/images/jobs/{job_id}`

插件安装位置:

```text
字字动画\_internal\plugins\image_plugins\flux2klein_multi_edit
```

字字配置:

- 进入 `图片模型管理 -> 自定义插件`。
- 当前插件选择 `flux2klein多图编辑`。
- 端点地址填写 FLUX.2 Klein 应用页面复制出来的地址, 例如 `https://...:8443/`。
- 模型可选:
  - `flux-2-klein-9b`
  - `flux-2-klein-4b`
- 输出宽高可直接设置, 例如 `1080 x 1920`。
- 图片映射最多可添加 8 张图, 但日常建议 1-4 张, 图越多越容易牺牲质量和一致性。

这条路线和 Zealman v8.51 的关系:

- Zealman v8.51 内部有 Flux / Flux2 / Klein 相关 ComfyUI 工作流。
- 但 `flux2klein_multi_edit` 插件主要面向 `FLUX.2-Klein-图像快速编辑` 这类专用应用接口。
- 只有当某个服务器页面明确提供 `/v1/images/jobs` 或 `/v1/images/edits` 这类 OpenAI 风格接口时, 才适合用这个插件。
- 如果只是在 Zealman v8.51 内打开普通 ComfyUI 工作流, 应走路线 A 或路线 B。

费用边界:

- 插件本体免费不等于算力免费。
- 如果单独启动 `FLUX.2-Klein-图像快速编辑` 应用实例, 仍然会产生对应 AutoDL/GPU 实例运行费用。
- 不要误以为它会调用 AutoDL 平台付费模型 API; 当前理解是它调用该应用实例本地服务, 主要费用仍是服务器实例费用。

## 是否需要上传到服务器

初步判断: 不需要把这套字字插件上传到服务器。

原因:

- `AutoDL-zealman-ImageAPI` 和 `AutoDL-zealman-VideoAPI` 是字字动画客户端插件。
- 插件应安装在本地字字动画软件目录下。
- 服务器端 Zealman v8.51 已经提供 API 生成、工作流导入和 `/api/workflow/generate` 调用能力。
- 插件通过网络调用 Zealman 管理面板 API, 不是 ComfyUI 自定义节点。

只有在服务器缺少 Zealman API 功能、或者需要把某些 API 工作流 JSON 同步到服务器面板时, 才考虑上传对应 JSON。当前服务器已是 Zealman v8.51, 暂不需要上传插件本体。

## 字字插件安装位置

图像插件:

```text
字字动画\_internal\plugins\image_plugins
```

视频插件:

```text
字字动画\_internal\plugins\video_plugins
```

对应复制:

```text
AutoDL-zealman-ImageAPI -> image_plugins
AutoDL-zealman-VideoAPI -> video_plugins
flux2klein_multi_edit -> image_plugins
```

复制后关闭并重启字字动画, 在自定义插件里选择对应插件。

## Zealman API 侧流程

1. 打开 AutoDL 暴露的 Zealman 管理面板, 也就是 `WebUI-6008`。
2. 确认 ComfyUI 已启动, 即 `6006` 后端正常。
3. 在 ComfyUI 里打开目标工作流, 用低参数确认能跑通。
4. 从 ComfyUI 菜单导出 API 格式 JSON。
5. 进入 Zealman 面板的 `API生成`。
6. 导入刚才保存的 API JSON。
7. 使用智能筛选, 勾选需要暴露给用户的字段。
8. 完成配置, 记录:
   - `workflow_id`
   - 正向提示词字段, 例如 `22:prompt`
   - 负向提示词字段, 例如 `2:prompt`
   - 种子字段, 例如 `15:seed`
   - 图片字段, 例如 `19:image`, `20:image`, `21:image`
   - 其他需要固定或自定义的字段

## 字字插件配置要点

Base URL:

- 填 Zealman 管理面板公网地址。
- 使用 AutoDL 的 `WebUI-6008` 地址。
- 不要填 ComfyUI 的 `6006` 地址。
- 这是自定义插件模式的规则; 如果使用字字内置 ComfyUI 工作流模式, 则应填 `WebUI-6006` 对应的 ComfyUI 地址。

Workflow ID:

- 填 Zealman API 生成页面里的 `workflow_id`。
- 示例: `F01-FireRed-Image-Edit-1`

提示词映射:

- 正向提示词键名: 根据 Zealman API 详情填, 例如 `22:prompt`
- 负向提示词键名: 根据 Zealman API 详情填, 例如 `2:prompt`

图片映射:

- 图像字段来自 `input_values`。
- 示例:
  - `19:image` -> 首帧
  - `20:image` -> 尾帧
  - `21:image` -> 参考图/图1

尺寸:

- 如果工作流尺寸由首图或参考图决定, 宽度/高度字段可以不填。
- 如果工作流明确暴露了尺寸字段, 再映射对应键名。

种子:

- 可以填种子字段, 例如 `15:seed`。
- 可开启每次随机种子。

轮询:

- 插件默认调用:
  - `POST /api/workflow/generate`
  - `GET /api/workflow/result?prompt_id=...`
- 默认轮询间隔可设为 2 秒。
- 视频任务最长等待时间建议更长, 例如 600 秒或更高。

## 多图占位规则

如果某个工作流固定要求 3 张图, 字字动画侧每次也要提供 3 张图。只想用 1 张或 2 张时, 可以用一个很小的空白 PNG 占位。

建议:

- 准备 10x10 或类似尺寸的白色/黑色 PNG。
- 放在不重要的尾帧或参考图位置。
- 目的只是满足工作流输入数量, 避免报错。

## 工作流字段改造要点

字字动画常用参数主要是:

- ComfyUI 服务器地址和连接测试。
- 正向提示词、负向提示词。
- 单图、双图、首尾帧、参考图。
- 视频输入。
- 宽度、高度。
- 时长或帧数。
- 种子和随机种子。
- 输出文件名前缀。

有些 Zealman 工作流是全自动提示词, 或提示词输入框是普通 `STRING` 节点, 字字动画可能识别不到可设置字段。处理方式:

1. 在 ComfyUI 中新增 `CR Prompt Text` 节点。
2. 将 `CR Prompt Text` 的输出接到原本需要提示词的输入口。
3. 用它替换字字无法识别的提示词输入框。
4. 再导出 API JSON。

文件名前缀建议:

- 不管输出是图片还是视频, 输出文件名前缀都改成简单英文字母, 例如 `comfyui`。
- 避免中文、特殊符号或过长前缀导致字字取结果不稳定。

时长设置:

- 有些视频工作流使用“秒”。
- 有些视频工作流使用“帧数”。
- 导入字字前需要确认节点到底控制的是秒数、帧数、FPS 还是最大长度。

已知需要优先整理映射的工作流:

- `G05-图生视频-Wan2.2+LightX2V最强动态`: 已整理到 `workflows/zizi_mappings/g05_wan22_lightx2v.md`
- `H27-LTX2.3首尾帧`: 已整理到 `workflows/zizi_mappings/h27_ltx23_first_last_frame.md`
- `H08-文生视频-LTX2.0多模态全自动16秒低配`: 已整理到 `workflows/zizi_mappings/h08_ltx20_t2v_multimodal_low.md`
- `Y06-图像改为下一个场景图`
- `H28-LTX2.3-高动态图生视频`
- `A01-文生图-Qwen2512`

## 常见故障排查

### 字字测试 ComfyUI 失败, `/object_info` 返回 404

常见原因:

- 填了 Zealman 管理面板地址, 而不是 ComfyUI 地址。
- 填了启动成功页地址, 但没有进入真实 ComfyUI 页面。
- 当前 AutoDL 映射地址不规范。

处理:

1. 回到 AutoDL 实例页。
2. 点击 `WebUI-6006`。
3. 等 ComfyUI 真正打开后, 复制浏览器地址栏里的地址。
4. 把该地址填到字字动画的 ComfyUI 外部服务器地址。
5. 重新测试, 需要看到 `/object_info` 和 WebSocket 都通过。

### 导入字字后看不到节点

常见原因:

- 导出的是普通工作流 JSON, 不是 API JSON。
- JSON 来自 Zealman 面板配置, 不是 ComfyUI 的 API 格式导出。

处理:

1. 回到 ComfyUI。
2. 打开目标工作流。
3. 通过 ComfyUI 菜单导出 API JSON。
4. 再导入字字动画。

### 工作流能在 ComfyUI 跑, 字字拿不到结果

可能原因:

- 部分节点和字字轮询结果不兼容。
- SeedVR2 等修复放大类工作流可能存在结果路径识别问题。
- 输出节点或文件名前缀不稳定。

处理:

- 先换简单输出前缀, 例如 `comfyui`。
- 先用 Zealman 面板或 ComfyUI 本身确认工作流能稳定输出。
- 字字仍不稳定时, 优先改走 Zealman 自定义 API 插件模式。

## 与当前项目的关系

这套插件适合把 Zealman 内置工作流变成字字动画可调用的生产入口。后续可以为每条已验证工作流维护一份映射表:

- Zealman 工作流名
- `workflow_id`
- Base URL 类型
- 输入字段映射
- 图片数量要求
- 输出类型
- 平均耗时
- 显存峰值
- 是否适合批量生产

## 下一步

1. 先在 Zealman 面板选 1 条图像工作流和 1 条视频工作流生成 API。
2. 记录对应字段映射。
3. 在字字动画本地插件里填入映射。
4. 用低参数小图/短视频测试。
5. 测通后, 再沉淀为项目工作流模板。
