<!-- Copilot instructions for contributors and AI coding agents -->
# 使用说明（针对 AI 辅助编码代理）

以下说明旨在让 AI 编码代理（或新来开发者）在本仓库中立刻上手。内容基于仓库真实代码与脚本，请只修改可被仓库文件验证或运行的部分。

## 项目一览（大局观）
- 本仓库为 MaaFramework 的项目模板（见 `README.md`），主要组成：
  - `agent/`：示例 agent 实现（`main.py`, `my_action.py`, `my_reco.py`），通过 `maa.agent.AgentServer` 注册自定义动作/识别器。
  - `assets/`：包含运行时资源（模型、pipeline、interface.json 等）；`assets/MaaCommonAssets` 为大体积子模块，必须存在以完成 OCR 功能。
  - `deps/`：放置 MaaFramework 二进制/共享库（release 包）——`install.py` 会读取此处进行打包。
  - `MFAAvalonia/`：桌面客户端资源（Windows UI/配置），与 agent/ 通信为运行时集成的一部分。

## 关键开发/运行流程（命令示例，PowerShell）
- 安装依赖（常见）：
```
pwsh> python -m pip install -r requirements.txt
pwsh> python -m pip install json-with-comments
```
- 配置 OCR 资源（会检查 `assets/MaaCommonAssets/OCR` 并拷贝默认模型到 `assets/resource/model/ocr`）：
```
pwsh> python .\configure.py
```
- 打包/安装到 `install/`（会检查 `deps`，并拷贝 agent、resource、interface.json 并写入版本）：
```
pwsh> python .\install.py v1.0.0
```
- 启动示例 agent（需要由上层进程传入 `socket_id`）：
```
pwsh> python .\agent\main.py <socket_id>
```
- 校验资源包（示例）：
```
pwsh> python .\check_resource.py <path-to-bundle-dir>
```

## 项目特有约定与模式（必须遵守的可发现规则）
- 自定义动作/识别器通过 `AgentServer` 装饰器注册：
  - `@AgentServer.custom_action("name")` -> 继承 `CustomAction` 并实现 `run()`（见 `agent/my_action.py`）。
  - `@AgentServer.custom_recognition("name")` -> 继承 `CustomRecognition` 并实现 `analyze()`（见 `agent/my_reco.py`）。
- 在识别/流水线中可使用 `context.run_recognition(...)` 或 `context.override_pipeline(...)` 来按需覆盖 pipeline 配置；也可用 `context.clone()` 创建局部上下文。示例在 `agent/my_reco.py`。
- `install.py` 会：
  - 要求 `deps/bin` 存在（MaaFramework release 解压后的目录）。
  - 拷贝 `deps/bin` 到 `install/`，并忽略特定控制单元文件（查看 `ignore_patterns`）。
  - 读取并更新 `install/interface.json` 的 `version` 字段为传入的版本参数。

## 集成与依赖点（外部/跨组件）
- 强依赖：`assets/MaaCommonAssets` 子模块（OCR 模型）——缺失会导致 OCR 报错。
- 二进制依赖：`deps/` 必须包含 MaaFramework 的 `bin` 与 `share/MaaAgentBinary`。
- Python 包：项目使用 `requirements.txt`（仓库根），且 `install.py` 明确依赖 `json-with-comments`（导入名 `jsonc`）。

## 调试与常见问题线索
- Windows 上运行本仓库示例应用可能需要 Visual C++ 运行库（`vc_redist.x64.exe`），详见 `README.md`。
- 日志与调试文件在 `debug/logs/`，可以在出现运行时错误时查阅。

## 可执行示例（快速代码片段以供 AI 参考）
- 注册一个自定义动作（直接参照 `agent/my_action.py`）：
```python
@AgentServer.custom_action("my_action_111")
class MyCustomAction(CustomAction):
    def run(self, context, argv):
        # 业务逻辑
        return True
```

## 编写补丁/变更时的注意点（给 AI 的具体约束）
- 只修改 `assets` 或 `agent` 下的示例实现时，应保持对 `interface.json` 和 `install.py` 的兼容性。不要更改 `install.py` 的输出结构，除非同时更新打包/发布流程。
- 若添加新的外部依赖，务必同时更新 `requirements.txt` 与 `README.md` 并在 `install.py` 中提供兼容性说明（若需要在 install 阶段读取）。

---
如果此文件有遗漏或你希望包含更多示例（例如 pipeline JSON 片段或 MaaFramework API 的常用调用），请告诉我需要补充的细节或你更关心的区域。 
