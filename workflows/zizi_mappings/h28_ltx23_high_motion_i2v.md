# H28 字字动画映射卡: LTX2.3 高动态图生视频

整理时间: 2026-05-23

用途: 将 Zealman ComfyUI 工作流 `H28-LTX2.3-高动态图生视频` 接入字字动画, 用一张首帧图片生成高动态短视频。

当前来源:

- 本地 API JSON: `D:\AI-Video-Workflow-Lab\字字工作流插件\可导入字字的API工作流\H28-LTX2.3-高动态图生视频.json`

## 推荐接入路线

使用路线 A: 字字直接连接 ComfyUI。

原因:

- 该工作流已有可直接导入字字动画的 API JSON。
- 这是单图图生视频工作流, 在字字的 `视频模型管理 -> ComfyUI工作流` 中导入。
- 输入一张首帧图, 再用提示词控制镜头和动作。

连接地址应使用 AutoDL `WebUI-6006` 打开的真实 ComfyUI 地址, 不是 Zealman 管理面板 `WebUI-6008`。

## 节点映射

| 节点 | 类型 | 字字中建议设置 | 说明 |
| --- | --- | --- | --- |
| `#269` | `LoadImage` | 首帧图片 | 输入起始画面/参考图。 |
| `#304` | `CLIPTextEncode` | 不替换 | 该节点接收 `#475 CR Prompt Text`, 不要直接改。 |
| `#315` | `CLIPTextEncode` | 负向提示词 | 默认负向词较短, 可按需补充。 |
| `#380` | `VHS_VideoCombine` | 输出文件名前缀保持 `comfyui` | 当前 JSON 已是 `comfyui`, 输出为 mp4。 |
| `#457` | `Int` | 宽度 | 示例值 `360`; 注意最终链路会放大, 不要设太大。 |
| `#459` | `Int` | 高度 | 示例值 `640`; 注意最终链路会放大, 不要设太大。 |
| `#467` | `Int` | 视频秒数 | 示例值 `5`。 |
| `#469` | `Float` | FPS | 示例值 `30`; 同时影响输出帧率和总帧数。 |
| `#475` | `CR Prompt Text` | 正向提示词 | 写镜头、动作、风格和动态描述。 |

## 当前 API JSON 中的关键默认值

- `#269 image`: `123.png`
- `#315 text`: `pc game, console game, video game, cartoon, childish, ugly`
- `#380 filename_prefix`: `comfyui`
- `#380 format`: `video/h264-mp4`
- `#457 Number`: `360`
- `#459 Number`: `640`
- `#467 Number`: `5`
- `#469 Number`: `30`
- `#475 prompt`: 英文电影感高动态视频提示词
- 关键模型:
  - `Ltx-2.3/ltx-2.3-22b-distilled_transformer_only_fp8_input_scaled_v3.safetensors`
  - `gemma_3_12B_it_fp8_e4m3fn.safetensors`
  - `ltx-2.3_text_projection_bf16.safetensors`

## 尺寸注意事项

这个工作流会先使用 `#334 ImageResizeKJv2` 把输入图 resize 到 `#457/#459`, 后续潜空间和解码链路会再放大。原教程提醒“生成的比这个尺寸要大 2 倍, 所以别设置太大”。

首轮建议:

- 竖屏: `#457 = 360`, `#459 = 640`
- 更清晰但更吃资源: `#457 = 512`, `#459 = 896`

不要一开始就填 1080 x 1920。48G 显存也要先跑小样例, 再逐步上调。

## 使用流程

1. 确认字字动画已连接 `WebUI-6006` 的 ComfyUI 地址, 且 `/object_info` 和 WebSocket 测试通过。
2. 在字字 `视频模型管理 -> ComfyUI工作流` 中导入 H28 API JSON。
3. 在素材区选择起始图片, 右键设置为 `首帧`。
4. 确认 `#269` 对应首帧图片。
5. 确认 `#304` 为不替换。
6. 将 `#475` 设置为正向提示词。
7. 将 `#315` 设置为负向提示词。
8. 设置尺寸:
   - `#457`: 宽度
   - `#459`: 高度
9. 设置时长和帧率:
   - `#467`: 秒数
   - `#469`: FPS
10. 确认 `#380` 文件名前缀为 `comfyui`。
11. 点击生成视频。

## 正向提示词建议

H28 适合高动态, 提示词要明确镜头和动作:

```text
Cinematic vertical video, ultra realistic.
The camera starts from a low-angle close-up, then quickly pushes forward.
The character turns her head toward the camera, hair and clothes moving in the wind.
Strong motion, dramatic lighting, smooth camera movement, detailed face, natural body motion, clean background.
```

中文也可以, 但这条默认提示词是英文结构, 后续建议优先用英文或中英混合测试动态稳定性。

## 参数建议

首轮低风险配置:

- `#457`: `360`
- `#459`: `640`
- `#467`: `5`
- `#469`: `30`
- 批量数量: `1`

如果输出不够清晰, 第二轮再尝试:

- `#457`: `512`
- `#459`: `896`

## 验收标准

- 字字 ComfyUI 测试通过。
- API JSON 导入后可见节点和可配置字段。
- 首帧图片进入 `#269`。
- 正向提示词进入 `#475`, 而不是直接改 `#304`。
- 负向提示词进入 `#315`。
- 宽高进入 `#457/#459`。
- 秒数和 FPS 进入 `#467/#469`。
- 输出文件名前缀为 `comfyui`。
- 生成结果为 mp4, 字字能拿到输出。

## 常见问题

### 正向提示词不生效

检查是否把正向提示词写到了 `#475 CR Prompt Text`。`#304` 接收 `#475`, 应保持不替换。

### 爆显存或速度很慢

先降低:

- `#457/#459` 尺寸
- `#467` 秒数
- 批量数量保持 1

不要一开始用 1080 x 1920。

### 画面动态太乱

减少提示词里的多段动作, 使用一个主动作和一个镜头运动。例如“镜头推进 + 角色转头”即可。

### 字字拿不到结果

检查 `#380 VHS_VideoCombine` 的 `filename_prefix` 是否为 `comfyui`, 输出格式是否为 `video/h264-mp4`。

## 后续记录项

首次实测后继续补充:

- 4090-48G 上 5 秒视频耗时
- 显存峰值
- 推荐竖屏尺寸
- 与 G05 Wan2.2 图生视频的动态、清晰度、稳定性对比
- 适合短剧/漫剧的高动态提示词模板
