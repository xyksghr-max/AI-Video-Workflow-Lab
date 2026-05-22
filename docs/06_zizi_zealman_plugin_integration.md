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
- `字字插件/image_plugins/bizyair_2fen_image/`

该目录是第三方下载包, 当前只作为本地参考和安装源, 不提交 Git。

## 这套插件解决什么问题

有些 ComfyUI 工作流可以在 Zealman 镜像里正常运行, 但直接导入字字动画时可能出现节点识别不正确。这个插件的思路是:

1. 在 Zealman 面板里把已跑通的 ComfyUI 工作流转换成 API。
2. 字字动画不再直接解析复杂工作流 JSON。
3. 字字动画插件只调用 Zealman 的工作流 API, 并把文本、图片、种子、尺寸等参数映射到 `input_values`。

这样可以绕过字字动画对某些 ComfyUI 节点识别不完整的问题。

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
