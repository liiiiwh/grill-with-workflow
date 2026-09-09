# Grill With Workflow + TDD

一套面向 AI 编程助手的工程技能：`grill-with-workflow` 基于 `grill-me` 技能，针对 workflow 的多 Agent 调度进行优化；搭配 `tdd`，形成从需求澄清、任务调度到测试验收的工作流程。

## 技能功能

### grill-with-workflow

基于 `grill-me` 的追问与设计澄清方式，增加面向 workflow 多 Agent 调度的协作规则。由主 Agent 负责项目治理、任务分配、监督与最终验收：

- **澄清需求**：必要时通过追问（grill）确认需求和设计中的关键决策。
- **沉淀项目知识**：按需维护 `CONTEXT.md`（领域术语）、`ARCHITECTURE.md`（模块边界与依赖）、`LOGIC.md`（流程、状态与业务规则）。
- **控制委派范围**：只有存在多个边界清晰、可独立执行的任务时才使用 SubAgents；简单或顺序任务由主 Agent 直接完成。
- **隔离职责**：避免文件和模块归属重叠；SubAgents 读取项目知识、执行任务并使用 TDD，冲突交回主 Agent。
- **监督与验收**：检查进展、阻塞、范围偏移和架构冲突，并依据需求、业务逻辑与测试验收结果。

随技能提供领域、架构、业务逻辑和 ADR 的文档格式参考。

### tdd

使用 **Red → Green → Refactor** 循环实现功能或修复问题：

1. 明确公共接口与需要验证的行为，结合项目术语和已有架构决策确定计划。
2. 先写一个失败测试，再写使它通过的最小实现，建立第一条完整行为路径。
3. 一次增加一个行为，逐步重复失败测试与最小实现。
4. 测试通过后再重构，每次重构后重新运行测试。

重点是通过公共接口验证行为，避免依赖内部实现的脆弱测试；仅在系统边界使用必要的 mock。随技能提供测试示例、mock 指南、可测试接口设计、深模块和重构参考。

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

安装后，在支持技能的编程助手中明确指定技能：

```text
使用 grill-with-workflow 实现订单取消功能，先梳理需求、领域术语和业务规则，再决定是否需要拆分任务。
```

```text
使用 tdd 为购物车增加优惠券功能，先确认关键行为，然后逐个完成 Red → Green → Refactor。
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
    │   └── LOGIC-FORMAT.md
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

两个技能及其参考文件从本地已有版本完整打包。`grill-with-workflow` 基于 `grill-me`，针对 workflow 多 Agent 调度优化，重点覆盖任务拆分、职责隔离、运行监督与结果验收。`grill-me` 与 `tdd` 的上游均来自 [Matt Pocock 的 skills](https://github.com/mattpocock/skills)。

上游内容保留 MIT 许可证及版权声明，详见 [grill-with-workflow 第三方声明](skills/grill-with-workflow/THIRD-PARTY-NOTICES.txt) 与 [tdd 第三方声明](skills/tdd/THIRD-PARTY-NOTICES.txt)。
