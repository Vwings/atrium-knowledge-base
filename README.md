# Atrium 知识库

Atrium 的产品知识库：产品探索、设计与决策。产品代码在同级的 `atrium/` 仓库；本仓库记录产品为什么这样设计。本文是唯一入口，探索进度只在这里维护。

## 目录

- `research/`：产品探索，按主题组织，随讨论更新；过时内容直接删除，历史靠 Git 留底。
- `product/`、`design/`、`decisions/`：确认后的产品定义、技术设计和 ADR，进入对应阶段时再建。

结论分四级：**已确认**（用户明确认可，下方汇总附原话或陈述）、**倾向**、**候选**、**否决**。Agent 不得把倾向自行升级为已确认。

## 文件索引

| 文件 | 内容 |
| --- | --- |
| [product-overview-draft.md](research/product-overview-draft.md) | 产品全貌草案，待用户整体评审 |
| [context-environment.md](research/context-environment.md) | 上下文环境（空间）：成员、项目来源、元数据与实例、生命周期、创建体验、`AGENTS.md` 生成 |
| [task-and-workflow.md](research/task-and-workflow.md) | 任务状态、任务与空间的关系、Task/Run 候选、工作流 |
| [extensions.md](research/extensions.md) | 通用插件机制与候选扩展点 |
| [agent-and-entries.md](research/agent-and-entries.md) | 基础定位、桌面与 CLI 形态、Agent 接入、内置 Agent、Agent 职责与语言路线 |
| [reference-architectures.md](research/reference-architectures.md) | Codex、OpenCode v2、Pi 程序架构参考，部署形态候选，技术路线待决问题 |

## 探索起因

用 AI Agent 开发时，一个任务常常涉及多个 Git 仓库。早期用 Atrium Skill 管理空间和任务时，每个空间只有一个当前任务；想把相对独立的任务分到别的空间并行推进，建空间仍要手工执行多条命令或反复委托 Agent，成本很高。

另一方面，现行 Skill 的任务状态、工作流和确认规则，多由 Agent 按自然语言规则执行，稳定性取决于模型是否理解和落实规则。产品探索要确认哪些能力应由程序维护和执行。

## 议题表

阶段 1 先讨论正向范围、再讨论排除范围：核心价值 → 能力地图 → 核心对象与关系 → 状态边界 → 入口关系与部署形态 → 非目标。前 7 项是阶段 1 的出口维度；用户确认这些维度足够收敛后进入产品定义收敛。

| 议题 | 结论分级 | 当前状态与位置 |
| --- | --- | --- |
| 产品目的与核心价值 | 已确认 | 上下文环境、任务状态、工作流三大能力；[产品全貌草案](research/product-overview-draft.md)待整体评审 |
| 能力地图 | 进行中 | 组织方式已确认（能力 + 贯穿部分）；能力内容见[产品全貌草案](research/product-overview-draft.md) |
| 核心对象与关系 | 进行中 | 一空间一 current、任务与空间同级已确认；Task/Run 是否分离为候选，见[任务与工作流](research/task-and-workflow.md) |
| 状态边界（结构化状态与 Markdown） | 倾向 | 元数据加磁盘实例、组合以元数据为准已确认，见[上下文环境](research/context-environment.md)；存储形式未选 |
| 入口关系（桌面、CLI、外部 Agent、内置 Agent） | 已确认（形态方向） | [Agent 与入口](research/agent-and-entries.md)；交付顺序未定 |
| 部署形态 | 候选 | [程序架构参考](research/reference-architectures.md)“单一可执行程序候选” |
| 非目标 | 已确认 | 6 条不做、3 条首期不做，见[产品全貌草案](research/product-overview-draft.md)“非目标” |
| 项目来源与接入方式 | 已确认 | [上下文环境](research/context-environment.md) |
| 创建体验（自然语言与可编辑资源清单） | 候选 | [上下文环境](research/context-environment.md)“创建体验” |
| 任务与空间的关系 | 已确认 | [任务与工作流](research/task-and-workflow.md) |
| 目标用户 | 已确认 | 不按个人或团队区分；见[产品全貌草案](research/product-overview-draft.md) |
| Agent 职责与外部 Agent 调度 | 已确认 | 内置 Agent 只做组织类工作，见[Agent 与入口](research/agent-and-entries.md) |
| 扩展机制 | 已确认（方向） | 通用插件机制，工作流是扩展点之一；机制细节为候选，见[扩展机制](research/extensions.md) |
| 授权与确认机制 | 候选 | [产品全貌草案](research/product-overview-draft.md)“贯穿所有能力的部分”；护栏的执行层机制另行研究 |
| 技术路线（语言、存储、协议、Agent runtime） | 候选 | [程序架构参考](research/reference-architectures.md)；阶段 4 再决策 |

当前下一步：用户判断阶段 1 能否出口、进入阶段 2 产品定义收敛。

## 已确认结论汇总

本节是已确认结论的唯一汇总位置。每条注明日期、用户原话或陈述，以及展开位置。一次确认只覆盖所列内容，不代表所在文档的其他候选内容获批。

| 日期 | 结论 | 用户原话或依据 | 展开位置 |
| --- | --- | --- | --- |
| 2026-09-22 | 基础定位：Atrium 是一套用于组织工作空间、管理任务和支持可插拔工作流的产品，以程序维护结构化状态，并提供相应的操作与可视化能力。CLI 和桌面端是产品组成，外部 Agent 是使用方。 | “我觉得这个定位可以” | [Agent 与入口](research/agent-and-entries.md) |
| 2026-09-22 | 项目来源与接入方式：裸仓库池建分支 worktree、固定提交的 detached 参考 worktree、共享目录池映射。 | 用户明确这是一直在用的实践，希望产品采用 | [上下文环境](research/context-environment.md) |
| 2026-09-23 | 桌面端提供图形操作与 Agent 自然语言交互；CLI 提供确定性命令与交互式 Agent 模式；所有入口共享核心；外部 Agent 通过确定性接口使用。Agent 从初期参与完整体验设计，交付顺序未定。 | “可以，我觉得可以” | [Agent 与入口](research/agent-and-entries.md) |
| 2026-10-20 | 产品核心是三块：上下文环境、任务状态追踪、工作流（作为扩展点）。 | 用户陈述：“核心就是在维护必要的上下文，让一个任务和他有关的上下文环境快速准备好，人机都友好”；“另外一个大的事，就是追踪任务状态”；“工作流呢，就是任务怎么执行，这也是一个扩展点” | [产品全貌草案](research/product-overview-draft.md) |
| 2026-10-20 | 一个空间一个 current，并行靠多空间；不直接用 worktree 是因为复杂任务涉及多个仓库。“空间等价于 worktree”只是类比。 | 用户陈述：“没打算改变一个空间一个 current 的模型，我想的是多空间来并行”；“复杂的系统往往是多个 git repo 组合的”；“这个只是概念的类比” | [上下文环境](research/context-environment.md) |
| 2026-10-20 | 目标是让开空间、开任务等成为随手的事；可靠性与授权是随手的前提，不是核心价值本身；一空间一 current 的理由是工作树隔离、上下文不含糊、授权范围明确、心智模型简单。 | “都认同” | [上下文环境](research/context-environment.md)、[任务与工作流](research/task-and-workflow.md) |
| 2026-10-20 | 集中存储一份空间元数据，磁盘上的空间是它的实例。 | “集中存储一份工作空间元数据，然后实际磁盘上的，可以实例化” | [上下文环境](research/context-environment.md) |
| 2026-10-20 | 组合以元数据为准，分支和改动实时读磁盘。 | “认同，组合以元数据为准，分支改动读磁盘” | [上下文环境](research/context-environment.md) |
| 2026-10-20 | 能力地图按“能力 + 贯穿所有能力的部分”两个维度组织，不分核心、支撑、扩展；工作流是能力之一。 | “可以” | [产品全貌草案](research/product-overview-draft.md) |
| 2026-10-20 | 知识库不作为空间的一等公民；空间成员只有“项目”一个通用概念，用途靠说明区分。 | 用户陈述：“他本质就是一个 git 仓库，或者文档目录，没什么特殊的吧，你可以就理解为是一个项目” | [上下文环境](research/context-environment.md) |
| 2026-10-20 | 早期工作方式资料只作背景，不作为全貌划分或产品结构的依据。 | 用户陈述：“不要过于使用旧资料影响现在的判断” | — |
| 2026-10-20 | Agent 接入作为贯穿问题，与入口、状态与操作接口、授权护栏并列。 | “可以，开始写产品全貌草案” | [Agent 与入口](research/agent-and-entries.md) |
| 2026-10-20 | 设计通用的插件机制，不局限于工作流；机制从一开始按通用方式设计，首期交付哪些扩展点划 MVP 时再定。 | “这个要设计一个通用的类似插件的机制，不要只局限在工作流”；对首期范围的推荐回复“同意” | [扩展机制](research/extensions.md) |
| 2026-10-20 | ~~先服务个人开发者~~（已被下一条取代）；工作流和常用项目组合做成可分享的文件。 | “其他三个也按你推荐的来” | [产品全貌草案](research/product-overview-draft.md) |
| 2026-10-20 | 不按个人或团队区分；不建立账号、角色和权限体系，也不做持续同步服务。 | “谁来干，这个不是我们主要考虑的东西” | [产品全貌草案](research/product-overview-draft.md) |
| 2026-10-20 | 跨机器恢复靠导出描述文件、另一台机器导入，各项目按 Git 地址拉取重建；优先级不高，细节以后展开。 | “产生一个描述文件，另一台机器直接 atrium import 这个文件，就可以恢复整个工作环境”；“优先级可能不是那么高” | [上下文环境](research/context-environment.md) |
| 2026-10-20 | 内置 Agent 只做组织类工作；业务编码交给外部 Agent，调度外部 Agent 暂不做。 | “其他三个也按你推荐的来” | [Agent 与入口](research/agent-and-entries.md) |
| 2026-10-20 | 任务与空间同级；任务记录项目需求和来源空间；内置 Agent 可登记任务；开始时三种去处，可按任务反向创建空间；开始后基本固定一个空间，仅暂停后原空间被占用时换空间。 | “其他三个也按你推荐的来”（用户此前追问任务的产生方式、项目依赖、反向创建空间、是否会用多个空间，结论据此收紧） | [任务与工作流](research/task-and-workflow.md) |
| 2026-10-20 | 非目标：不做编码 Agent、编辑器与 Git 日常操作、知识库与团队规则内容管理、通用 Agent 平台、代码备份、严格期望状态收敛；首期不做调度外部 Agent、以一次性单仓库小改动为首要场景。回收和重建前存在未提交或未推送改动时直接拦住。（第 7 条后改为不做账号角色权限体系与持续同步服务，见上方“不按个人或团队区分”一条。） | “同意，第5条改成拦住” | [产品全貌草案](research/product-overview-draft.md) |
| 2026-10-20 | 默认工作流以官方工作流包（普通插件，默认启用，可停用替换）提供；spec、Review 是工作流产物，挂在任务上，核心不写死；任务可以不使用工作流，但必须由用户明确选择。 | “同意，两个都按你推荐的来” | [任务与工作流](research/task-and-workflow.md) |

否决：

| 日期 | 结论 | 用户原话 | 展开位置 |
| --- | --- | --- | --- |
| 2026-10-20 | 把空间目录本身做成 Git 仓库。 | “你这么一分析，似乎把空间本身作为仓库过于粗暴了，不合适” | [上下文环境](research/context-environment.md) |

## 仍待确定

Task/Run 是否分离、插件机制细节与首期扩展点、权限与授权机制、交付阶段及验收。Rust、TypeScript、Pi、Electron、存储、协议和进程部署均未选定。先完成全貌，再划分阶段和 MVP。
