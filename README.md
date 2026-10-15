# Doc Auditor · 文档审校

一套中文 Agent Skill：检查技术文档中的语义歧义、逻辑缺口、指标口径、验收标准和沟通风险，并给出按严重程度排列的问题与修改建议。

## 适用场景

- 技术方案、架构方案、接口方案评审。
- 压测方案、测试计划、验收与上线文档检查。
- 汇报、复盘、PR 说明的语义和逻辑审校。

默认输出审校发现；用户要求“只检查、不新增”时收窄范围。只有用户明确要求修改时，才改写原文。技能无法仅凭文字证明线上状态、数据或方案正确，也不替代测试和人工评审。

## 目录

```text
doc-auditor/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── review-rubric.md
│   └── output-formats.md
├── README.md
├── LICENSE
└── .gitignore
```

入口为 [SKILL.md](SKILL.md)。[审校清单](references/review-rubric.md)覆盖多种文档类型；[输出格式](references/output-formats.md)提供问题表和措辞模板。`agents/openai.yaml` 提供客户端展示信息和调用策略。

## 本地安装

下载或克隆本仓库后，在仓库根目录运行以下命令（macOS / Linux，Bash 或 Zsh）：

```bash
(
  set -eu
  skill_dir="$HOME/.agents/skills/doc-auditor"
  if [ -e "$skill_dir" ] || [ -L "$skill_dir" ]; then
    echo "技能目录已存在，请先检查已有版本：$skill_dir"
    exit 1
  fi
  mkdir -p "$skill_dir"
  cp SKILL.md LICENSE "$skill_dir/"
  cp -R agents references "$skill_dir/"
)
```

这会安装到当前用户的 `~/.agents/skills/doc-auditor/`。如果已有同名技能装在 `~/.codex/skills/`，先检查是否为已有安装或软链接，避免重复安装。若技能列表未更新，可重启 Codex。

## 使用示例

在 Codex 中输入 `$` 选择技能，或直接发送：

```text
$doc-auditor 请检查以下虚构压测方案，只指出问题，不新增方案：
“启动 100 个并发，发送 1 万次请求。平均耗时低于 200 ms 即通过。
失败请求不计入平均耗时，成功率尽量高。”
```

当要求“只检查、不新增”时，典型输出为：

| 原文 | 问题 | 建议替换 |
|------|------|----------|
| 成功率尽量高 | 缺少明确通过线，无法判定是否验收通过 | 成功率达到事先约定的验收阈值；具体数值待确认 |

普通审校使用“位置、级别、问题、为什么有风险、建议改法”表。分级为 P0 阻塞、P1 重要、P2 清晰度、P3 格式；不确定事实应标注待核实。

## 兼容性与数据边界

这是遵循 `SKILL.md` 目录约定的文本技能，当前版本没有可执行脚本、MCP 配置或网络上传代码。此前已在本机 Codex 技能加载检查中验证技能可被识别；不同模型的实际输出可能不同。

Codex 桌面端与 CLI 可通过本地技能目录发现技能。ChatGPT 桌面端的可用性取决于客户端功能和技能发现配置；本仓库尚未验证其端到端调用。复制本地文件不会自动同步到 ChatGPT 网页版或手机端。

技能运行时，宿主模型或工具可能处理用户提供或授权读取的材料。请遵守所在组织的数据与模型使用政策。仓库的示例为虚构场景，提交 Issue、截图或改进示例时也请勿附带真实公司文档、客户信息、凭据或内部链接。

## 修改与贡献

直接修改 `SKILL.md` 或其引用文件即可调整工作流。提交修改时，请说明适用场景和预期行为，检查相对引用是否有效，并使用虚构材料验证。新增文件前请检查内容和 Git 暂存区；`.gitignore` 不能识别 Markdown 正文中的敏感信息。

## 许可证

本项目采用 [MIT License](LICENSE)，允许在保留许可要求的前提下使用、修改和分发，包括商业用途。技能输出需要人工复核，项目按许可证中的 “AS IS” 条款提供。
