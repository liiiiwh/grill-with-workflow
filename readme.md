# Grill With Workflow + TDD

一套面向 AI 编程助手的工程技能：`grill-with-workflow` 基于 `grill-me` 技能，针对 workflow 的多 Agent 调度进行优化；搭配 `tdd`，形成从需求澄清、任务调度到测试验收的工作流程。

## 技能功能

### grill-with-workflow

基于 `grill-me`，协调从需求澄清到多 Agent 实现与验收的完整流程。

- **Grill 追问**：逐项澄清需求、设计与验收标准。
- **文档先行**：自动读取、补齐并更新术语、架构和逻辑文档，再列任务。
- **TDD 实现**：主 Agent 与 SubAgents 均须调用 `tdd` 技能。
- **并行调度**：独立任务交给 SubAgents，明确职责并持续监督。
- **数量上限**：每个需求累计最多启动 5 个 SubAgent，仅用户明确允许无限使用时解除。
- **错误可见**：错误传递到用户交互层，禁止吞错或伪装成功。
- **复用与解耦**：优先扩展现有功能，禁止跨模块重复实现。
- **向下兼容**：保护旧接口与调用方，验证新旧行为。
- **拒绝兜底堆积**：修复根因，禁止无依据兜底和无限重试。
- **清理冗余**：扫描死代码、重复逻辑、不必要的深层循环与耦合。
- **强制收尾**：追加第 N+1 个清理任务，同步文档并验证结果。

项目文档：`CONTEXT.md`、`ARCHITECTURE.md`、`LOGIC.md`。

执行流程：**Grill → 更新文档 → 列任务 → TDD / SubAgents → 清理与验收**。

详细规则：[技能入口](skills/grill-with-workflow/SKILL.md) · [代码质量](skills/grill-with-workflow/references/code-quality.md) · [SubAgent 协作](skills/grill-with-workflow/references/subagents.md)。

### tdd

- **逐步实现**：一次一个行为，执行 Red → Green → Refactor。
- **行为测试**：通过公共接口验证，避免绑定内部实现。
- **边界 Mock**：只在必要的系统边界模拟依赖。
- **安全重构**：测试通过后重构，再运行验证。

## 安装

需要 Node.js 和 npm（提供 `npx`），并能够访问 GitHub。使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装。

### 交互式安装

在目标项目目录运行，按提示选择技能和 Agent：

```bash
npx skills add liiiiwh/grill-with-workflow
```

### 一次安装两个技能

以下命令将两个技能安装到当前项目，目标为 Claude Code：

```bash
npx skills add liiiiwh/grill-with-workflow --skill grill-with-workflow tdd --agent claude-code --yes
```

安装到用户目录，让所有项目可用：

```bash
npx skills add liiiiwh/grill-with-workflow --skill grill-with-workflow tdd --agent claude-code --global --yes
```

也可以将 `--agent claude-code` 替换为 `--agent codex`，或使用 `--agent claude-code codex` 同时安装到两个 Agent。

### 单独安装

```bash
npx skills add liiiiwh/grill-with-workflow --skill grill-with-workflow
npx skills add liiiiwh/grill-with-workflow --skill tdd
```

### 查看可安装技能

```bash
npx skills add liiiiwh/grill-with-workflow --list
```

本仓库采用 Skills CLI 可识别的 `skills/<技能名>/SKILL.md` 结构，可直接从 GitHub 安装，无需发布 npm 包。

## 使用示例

安装后，在支持斜杠技能命令的编程助手中输入：

```text
/grill-with-workflow 初始化当前项目的知识文档。
```

```text
/grill-with-workflow 实现订单取消功能。
```

调用技能后会自动执行文档读取、缺失文档创建及更新流程，无需每次重复指定三个文件名。

```text
/tdd 为购物车增加优惠券功能。
```

组合使用时，`grill-with-workflow` 负责整体协调，`tdd` 负责具体实现的测试循环。

`grill-with-workflow` 的原始指令面向 Claude Code。技能中的 grill、workflow、subagent 和 loop 描述执行方式；本仓库仅包含这两个技能，不附带同名工具、其他技能或 Agent 运行时。委派和监督需要宿主提供相应能力；安装到其他 Agent 后，应按其实际能力使用。

## 目录结构

```text
.
├── LICENSE
├── readme.md
└── skills/
    ├── grill-with-workflow/
    │   ├── SKILL.md
    │   ├── LICENSE
    │   ├── THIRD-PARTY-NOTICES.txt
    │   ├── ADR-FORMAT.md
    │   ├── ARCHITECTURE-FORMAT.md
    │   ├── CONTEXT-FORMAT.md
    │   ├── LOGIC-FORMAT.md
    │   └── references/
    │       ├── code-quality.md
    │       └── subagents.md
    └── tdd/
        ├── SKILL.md
        ├── LICENSE
        ├── THIRD-PARTY-NOTICES.txt
        ├── deep-modules.md
        ├── interface-design.md
        ├── mocking.md
        ├── refactoring.md
        └── tests.md
```

## 许可证与来源

本项目采用 [Apache License 2.0](LICENSE)。每个技能目录附带许可证，方便单独安装和分发。

两个技能及其参考文件最初从本地已有版本打包，后续在本仓库迭代维护。`grill-with-workflow` 基于 `grill-me`，针对 workflow 多 Agent 调度优化，重点覆盖文档先行、任务拆分、职责隔离、运行监督、收尾清理与结果验收。`grill-me` 与 `tdd` 的上游均来自 [Matt Pocock 的 skills](https://github.com/mattpocock/skills)。

上游内容保留 MIT 许可证及版权声明，详见 [grill-with-workflow 第三方声明](skills/grill-with-workflow/THIRD-PARTY-NOTICES.txt) 与 [tdd 第三方声明](skills/tdd/THIRD-PARTY-NOTICES.txt)。
