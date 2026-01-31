# java-man

面向 Ops/SRE 的 Java 项目“商业级剖析”技能（Codex skill）。以业务流为先，聚焦集成、可运维性与 K8s 准备度，输出可交付报告。

## 能力与产出

- 生成两个报告文件：`PROJECT_PROFILE.md` 与 `DEPLOYMENT_RUNBOOK.md`
- 报告目录：`analyze-report-YYYY-MM-DD/`（自动按日期生成）
- 证据链驱动：每个非平凡结论都要求 Evidence Anchor，缺失则标记 Unknown
- 先业务后运维：业务能力与关键流程 → 集成矩阵 → 配置/数据层 → 部署与可观测

## 目录结构

- `skills/java-man/SKILL.md`：技能定义与完整工作流
- `skills/java-man/references/`：报告模板与表格（集成矩阵、配置清单、可视化图表、就绪度评分等）

## 在 Codex 中使用

1. 将 `skills/java-man` 放入 Codex 可发现的技能目录，例如：
    - `~/.codex/skills/java-man`（默认的用户级目录）
    - `./.codex/skills/java-man`（仓库级目录）
2. 重启 Codex 让技能被扫描加载
3. 显式调用：在对话中输入 `/skills` 选择技能，或直接在提示词里输入 `$java-man`
4. 也可让 Codex 隐式触发（当任务与技能描述匹配时）

示例提示：

```
$java-man
请为当前仓库生成 PROJECT_PROFILE.md 和 DEPLOYMENT_RUNBOOK.md
```

## 使用 npx 安装 / 更新（Skills Registry）

如果你希望从公开的技能目录安装（或更新）技能，可使用 skills registry 的 npx 命令：

```
# 列出可安装技能（category/slug）
npx codex-skills-registry@latest --list

# 安装或更新某个技能（重复执行即更新）
npx codex-skills-registry@latest --skill=category/skill-name --yes
```

skills registry 提供“一行安装”，并明确说明重复执行即可更新现有安装。 citeturn2view0

注意：本仓库如果尚未发布到 registry，则建议使用上面的“本地放置/仓库内放置”方式。

## 学习与参考（推荐）

- OpenAI 官方技能文档：技能结构、显式/隐式调用、技能目录位置等说明 citeturn1view0
- OpenAI 的技能示例仓库（skills catalog） citeturn0view1
- Skills Registry 目录（便于快速浏览与安装） citeturn2view0

## Version

0.1
