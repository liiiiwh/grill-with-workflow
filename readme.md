# Grill With Workflow + TDD

一套面向 AI 编程助手的工程技能：`grill-with-workflow` 基于 `grill-me` 技能，针对 workflow 的多 Agent 调度进行优化；搭配 `tdd`，形成从需求澄清、任务调度到测试验收的工作流程。

## 技能功能

### grill-with-workflow

基于 `grill-me` 的追问与设计澄清方式，增加面向 workflow 多 Agent 调度的协作规则。由主 Agent 负责项目治理、任务分配、监督与最终验收：

- **Grill 澄清需求**：沿设计分支逐项追问，先自行查代码能回答的问题；需要用户决策时，一次问一个问题并给出建议及取舍，直到关键行为、接口、边界和验收标准达成共识。
- **文档先行**：每次接受需求，先自动读取 `CONTEXT.md`（领域术语）、`ARCHITECTURE.md`（模块边界与依赖）、`LOGIC.md`（流程、状态与业务规则）；缺失时根据代码、配置和测试创建。核实并更新相关内容后，才列任务、进入 TDD 或委派 SubAgents。
- **扫描代码质量**：检查本次需求涉及的代码与直接依赖，识别死代码、重复逻辑、冗余兜底、深层循环与条件嵌套、循环依赖和职责耦合。
- **错误对用户可见**：所有错误沿调用链传递到顶层 UI、CLI、API 或任务状态；禁止只写日志、吞异常、返回空数据或伪装成功。底层模块传递结构化错误，由交互层统一展示；恢复后的错误也以恢复/警告状态反馈，可合并同源错误，避免逐层重复弹出。
- **先复用，再扩展**：编码前查找已有实现与调用方，同一能力由一个模块负责，禁止在不同模块重写。通过公共接口复用或迭代现有功能，避免循环依赖、跨模块访问内部实现、不必要的深层循环及重复抽象。
- **保持向下兼容**：扩展功能时保护现有调用方的接口和可观察行为，验证旧用法与新能力；必要兼容适配仍调用同一实现。确需破坏兼容时，先通过 grill 明确用户决策与迁移方案。
- **编码时防止兜底堆积**：新增兜底必须有实际失败证据或明确契约，定义责任层、触发条件、恢复行为及验证方式；禁止用空值、吞异常或备用路径掩盖根因。重试必须有次数/时间边界，并检查跨层重试是否放大请求。同一问题连续两次修复无效，停止叠加补丁，回到最小复现重新诊断。
- **选择执行方式**：每个实现任务明确采用“主 Agent + tdd 技能”或“SubAgent + tdd 技能”。执行者必须实际读取并遵循 `tdd` 技能，逐个行为完成 Red → Green → Refactor；委派不能免除 TDD。
- **控制委派范围**：每轮评估是否适合委派；宿主支持且存在多个边界清晰、可独立执行、并行有收益的任务时使用 SubAgents；简单或顺序任务由主 Agent 调用 `tdd` 完成。
- **隔离职责**：避免文件和模块归属重叠；SubAgents 读取项目知识、执行任务并使用 TDD，冲突交回主 Agent。
- **监督与验收**：检查进展、阻塞、范围偏移和架构冲突，并依据需求、业务逻辑与测试验收结果。
- **强制收尾任务**：N 个实现任务后追加第 N+1 个“清理、文档同步与验证”任务，等待全部实现整合后执行。清理本轮引入及影响范围内确认的问题，重新验证行为并同步三份文档。

随技能提供领域、架构、业务逻辑和 ADR 的文档格式参考。

入口 `SKILL.md` 保留核心流程和强制规则，详细检查按执行阶段读取：代码扫描、列任务和编码前必须读取 [代码质量契约](skills/grill-with-workflow/references/code-quality.md)，委派前主 Agent 与执行前的 SubAgents 必须读取 [协作规则](skills/grill-with-workflow/references/subagents.md)。参考文件随技能一起安装，不是可选建议。

每轮执行顺序：**读取文档与代码 → 澄清需求并扫描代码质量 → 更新文档 → 列任务 → TDD / SubAgent 实现 → 整合与清理 → 文档同步及验证验收**。

Grill、调用 `tdd` 技能和 SubAgent 调度是核心能力，文档与清理贯穿这些步骤。建议同时安装本仓库的两个技能；每位执行者都必须能读取 `tdd`，不能仅口头承诺使用 TDD。纯文档任务不强行编写代码测试。宿主缺少 SubAgent 能力时明确说明限制，由主 Agent 调用 `tdd` 执行。

文档明确区分当前实现与已确定但尚未实现的变更；执行中发现术语、架构或逻辑变化，由主 Agent 及时更新，再调整后续任务。清理以代码和行为证据为依据，保留必要的错误处理与兼容逻辑，避免无关重构，也不把用户未提交的改动视为垃圾。

防止兜底堆积的规则同时约束主 Agent 和 SubAgents，并纳入任务交付检查。即使正常路径测试通过，无依据的兜底和已失效的补丁仍需清理；必要的恢复逻辑要验证失败分支和重试耗尽后的行为。这些是技能中的执行与验收要求，并非运行时自动拦截机制。

错误处理要覆盖异步任务、部分完成和恢复场景，并保留原因链；面向用户展示可理解的信息，避免直接泄露内部堆栈或敏感数据。如果现有错误契约与新要求冲突，先澄清兼容方案，不能通过隐藏错误或静默破坏旧接口解决。

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
使用 grill-with-workflow，初始化当前项目的知识文档。
```

```text
使用 grill-with-workflow 实现订单取消功能，先梳理需求、领域术语和业务规则，再决定是否需要拆分任务。
```

调用技能后会自动执行文档读取、缺失文档创建及更新流程，无需每次重复指定三个文件名。

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
