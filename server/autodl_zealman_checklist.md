# AutoDL Zealman ComfyUI 检查清单

## 当前候选实例

- 平台: AutoDL
- 应用: zealman-ComfyUI
- 版本: v8.51
- 作者: zealman
- 区域: 西北B区
- 计费方式: 按量计费
- 候选 GPU: 4090-48G
- 参考价格: 约 ¥2.88/小时/每 GPU
- 参考资源: vCPU 12~22 核, 内存 64~100 GB
- 应用提示: 4090 / 5090 / 6000 / H800 可用

说明: 以上信息来自租机前截图, 最终以创建实例后的实际硬件、驱动、磁盘和应用目录为准。

## 第一批目标模型/能力

Wan2.2:
- Wan2.2-I2V-A14B FP8
- Wan2.2-Animate-14B FP8
- Wan2.2-T2V-A14B FP8
- Wan2.2 Fun Control FP8
- Wan2.2 Fun Inp / FLF2V FP8

LTX-2.3:
- LTX-2.3 22B dev FP8
- `ltx-2.3-22b-dev-fp8.safetensors`
- `gemma_3_12B_it_fp4_mixed.safetensors`
- `ltx-2.3-spatial-upscaler-x2-1.0.safetensors`

状态: 候选验证。租机后先检查是否已内置, 未内置时再核验来源、体积、许可证、目标目录和下载方式。

## 登录后先确认

- 当前用户和工作目录
- 系统盘和数据盘空间
- GPU 型号、显存、驱动、CUDA
- Zealman ComfyUI 根目录
- ComfyUI 启动命令和端口
- 已安装 custom nodes
- 已有模型列表
- `input`, `output`, `temp`, `models` 实际位置

## 禁止直接做的事

- 不直接清理系统目录。
- 不删除未知文件。
- 不重置 Zealman 应用。
- 不把大模型下载到系统盘。
- 不把服务器密码或 API key 写入 Git。

## 第一轮只做的小测试

- 使用已有模型和已有工作流。
- 分辨率、帧数、步数都用低参数。
- 只验证 ComfyUI 是否能正常执行和输出。
- 记录错误信息和输出路径。
