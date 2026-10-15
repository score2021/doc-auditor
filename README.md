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

## 从 GitHub 下载并安装到 Codex

以下命令适用于 macOS / Linux 的 Bash 或 Zsh。请先安装 Git 和 Codex CLI，并确认已经登录：

```bash
git --version
codex --version
codex login status
```

若尚未登录，运行 `codex login` 并完成登录。CLI 安装及账号要求见 [Codex CLI 官方说明](https://learn.chatgpt.com/docs/cli)。

### 下载与安装（整段复制到终端）

这段命令会从公开 GitHub 仓库克隆当前版本，只复制技能需要的文件和许可证，并在退出时删除下载临时目录；不会上传本地项目。已存在同名安装时会停止，请先检查已有版本。

```bash
(
  set -eu
  command -v git >/dev/null
  command -v codex >/dev/null
  skill_dir="$HOME/.agents/skills/doc-auditor"
  legacy_skill_dir="${CODEX_HOME:-$HOME/.codex}/skills/doc-auditor"
  if [ -e "$skill_dir" ] || [ -L "$skill_dir" ] || [ -e "$legacy_skill_dir" ] || [ -L "$legacy_skill_dir" ]; then
    echo "发现同名技能，请先检查已有安装：$skill_dir 或 $legacy_skill_dir"
    exit 1
  fi
  download_dir="$(mktemp -d)"
  trap 'rm -rf "$download_dir"' EXIT
  git clone --depth 1 https://github.com/score2021/doc-auditor.git "$download_dir/doc-auditor"
  mkdir -p "$skill_dir"
  cp "$download_dir/doc-auditor/SKILL.md" "$download_dir/doc-auditor/LICENSE" "$skill_dir/"
  cp -R "$download_dir/doc-auditor/agents" "$skill_dir/"
  cp -R "$download_dir/doc-auditor/references" "$skill_dir/"
  test -f "$skill_dir/SKILL.md"
  echo "安装完成：$skill_dir"
)
```

技能安装到当前用户的 `~/.agents/skills/doc-auditor/`，可在不同项目中使用。[Codex 官方技能文档](https://learn.chatgpt.com/docs/build-skills)说明了该用户级发现路径。不要把它复制成 `~/.agents/skills/doc-auditor/doc-auditor/SKILL.md` 这样的多层目录。

### 在 Codex 中调用

安装后新开一个 Codex 会话。CLI 运行 `codex`，输入 `/skills` 查看已发现的技能，或输入 `$` 搜索 `doc-auditor`；桌面端在新会话中输入 `$` 选择该技能。未出现时重启客户端。

也可以直接在终端执行以下完整命令进行一次真实调用。命令使用单引号保留 `$` 技能标记；`--skip-git-repo-check` 允许在非 Git 目录演示，`read-only` 限制本地文件写入。

```bash
codex exec --skip-git-repo-check --sandbox read-only '$doc-auditor 请检查以下虚构压测方案，只指出问题，不新增方案，也不修改文件：“启动100个并发，发送1万次请求。平均耗时低于200ms即通过。失败请求不计入平均耗时，成功率尽量高。”'
```

正常结果应包含审校问题表，并指出成功率缺少阈值及失败请求排除造成的指标偏差；不应凭空填写验收数值。只看到“安装完成”还不能证明调用成功，需要检查 Codex 的实际输出。

### 常见问题

- `codex: command not found`：先安装 Codex CLI，或修复终端 `PATH`。
- 登录失败、模型无权限、额度不足或网络错误：先解决 Codex 账号和连接问题；安装技能本身不会提供模型访问权限。
- 找不到技能：检查 `~/.agents/skills/doc-auditor/SKILL.md` 存在，并重启 Codex；检查配置中是否禁用了该技能。
- 提示同名技能已存在：检查 `~/.agents/skills/` 和 `${CODEX_HOME:-~/.codex}/skills/` 的已有目录或软链接，再决定保留哪个版本；不要直接覆盖未知文件。
- GitHub 克隆失败：检查 GitHub 的 HTTPS 连接；安装命令会清理下载临时目录。若复制阶段中断，先检查目标目录是否完整，再决定是否重新安装。

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

这是遵循 `SKILL.md` 目录约定的文本技能，当前版本没有可执行脚本、MCP 配置或网络上传代码。已在 macOS、Codex CLI 0.162.1 环境完成从公开 GitHub 克隆、按本文命令安装、隔离技能发现到真实 `codex exec` 调用的验证；Bash 与 Zsh 安装命令均通过。验证只使用新下载的技能，未借用已有同名安装。不同系统、客户端版本、账号和模型环境仍需自行确认。

Codex 桌面端与 CLI 可通过本地技能目录发现技能。ChatGPT 桌面端的可用性取决于客户端功能和技能发现配置；本仓库尚未验证其端到端调用。复制本地文件不会自动同步到 ChatGPT 网页版或手机端。

技能运行时，宿主模型或工具可能处理用户提供或授权读取的材料。请遵守所在组织的数据与模型使用政策。仓库的示例为虚构场景，提交 Issue、截图或改进示例时也请勿附带真实公司文档、客户信息、凭据或内部链接。

## 修改与贡献

直接修改 `SKILL.md` 或其引用文件即可调整工作流。提交修改时，请说明适用场景和预期行为，检查相对引用是否有效，并使用虚构材料验证。新增文件前请检查内容和 Git 暂存区；`.gitignore` 不能识别 Markdown 正文中的敏感信息。

## 许可证

本项目采用 [MIT License](LICENSE)，允许在保留许可要求的前提下使用、修改和分发，包括商业用途。技能输出需要人工复核，项目按许可证中的 “AS IS” 条款提供。
