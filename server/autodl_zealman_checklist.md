# AutoDL Zealman ComfyUI 检查清单

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
