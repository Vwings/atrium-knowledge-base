# 程序架构参考

## 状态

研究输入。记录 2026-09-22 对 Codex、OpenCode v2、Pi 程序架构的只读调查，以及由此形成的部署形态候选。这里只提供工程模式和需要提前约束的非功能问题，不是 Atrium 的产品模板，也没有形成技术选型或 Accepted ADR。

使用边界：

- 先从 Atrium 自身确认用户场景、核心对象、状态不变量、入口关系和 MVP，再把下列机制映射到已确认的需求上，不从参考项目反推产品模块。
- 参考的作用只有三种：提供可行的工程模式（多入口共享核心、协议生成、事件投影、插件生命周期、客户端恢复）；识别需要提前约束的非功能问题（幂等、恢复、背压、版本、权限边界）；验证候选方案的复杂度和代价。
- 早期 `atrium/` 只有少量提交，曾探索 Rust 核心、CLI、GUI、Agent 和 Skill，后来又考虑 JavaScript、Electron 和 Pi；Pi 从未实际使用。这些都不构成技术约束。

## 技术路线待决问题

阶段 4 再决策，当前均未选定：

1. CLI、服务和桌面端的交付顺序，以及是否采用单一可执行程序。
2. 核心实现语言（Rust、TypeScript 或混合）；核心的语言不决定 Agent 的语言，见 [Agent 与入口](agent-and-entries.md)。
3. 状态存储采用 SQLite、结构化文件或组合方案；Markdown 如何与结构化状态关联。
4. 工作流插件的形态：声明式定义、规则脚本、动态库、独立进程或多种级别。
5. Skill 是独立适配层、随 CLI 分发，还是由插件提供。
6. 是否需要任务运行会话，以及它与工作流执行实例、Agent 对话的关系。

## 单一可执行程序候选（部署形态）

参考 Codex 的程序组织方式，Atrium 可以由一个可执行程序提供多个运行模式：

```text
atrium                 # 普通 CLI，一次性操作或脚本调用
atrium server          # 服务模式，等待桌面端或其他客户端连接
atrium ... --embedded  # 进程内复用核心，不另起服务进程
```

这表示共享核心能力、状态模型和协议边界，而不表示桌面端必须与 CLI 合并为同一个进程，也不表示服务必须常驻。服务是否独立运行、CLI 是否自动启动服务、桌面端如何发现服务，均属于后续决策。

候选分层如下：

```text
CLI / Desktop / 外部 Agent（由 Skill 指导调用）
             ↓
     Atrium protocol and API
             ↓
   Atrium runtime and state store
             ↓
 Workspace / task / workflow operations
```

这里的“服务”是程序部署方式，不是 Atrium 的业务定位。它可以在本地进程内或独立进程中承载相同的请求处理能力。

## Codex

本轮将 `codex/` 作为只读参考，基线为 `064e701b0fa4f5c0dcaf195a565d088f99bb6af8`。参考目标是程序设计，而不是 Codex 的 Agent 产品功能。

阅读源码前先核实了官方文档 <https://developers.openai.com/codex/app-server/>：客户端通过协议使用会话、审批和事件流，职责划分与源码阅读一致。官方文档不足以断言 ChatGPT 桌面端的内部架构，本文不作此推断。

### 值得参考的机制

- **一个程序、多种运行路径**：App Server 既有独立传输入口，也有进程内宿主；公共能力不绑定单一部署方式。
- **协议优先**：请求、响应和事件由 Rust 类型定义，并生成 JSON Schema / TypeScript；稳定接口和实验接口有明确边界。
- **客户端适配层**：进程内客户端与远程客户端收敛到相近的请求、通知和事件接口。
- **运行时与持久化解耦**：运行中的会话对象、连接订阅和持久化记录分别管理；稳定 ID 连接不同层次。
- **恢复顺序和事件一致性**：恢复历史时协调已有记录与实时事件订阅，避免重连过程漏事件。
- **生命周期与闲置卸载分离**：客户端断开、运行对象卸载、历史归档或删除是不同操作。
- **事件队列和背压**：请求通道和事件消费分别处理，显式处理队列饱和和客户端不消费的情况。
- **能力协商和版本控制**：客户端在初始化时声明能力，实验接口需要显式启用；工作流插件可以借鉴这一机制。

以上机制的主要阅读位置（路径相对 `codex/`）：

- 进程内宿主与客户端适配：`codex-rs/app-server/src/in_process.rs`、`codex-rs/app-server-client/src/lib.rs`
- 协议类型生成与稳定/实验边界：`codex-rs/app-server-protocol/src/precomputed_exports.rs`、`codex-rs/app-server-protocol/src/protocol/common.rs`、`codex-rs/app-server-protocol/src/experimental_api.rs`
- 会话运行对象、订阅与持久化：`codex-rs/core/src/thread_manager.rs`、`codex-rs/app-server/src/thread_state.rs`、`codex-rs/thread-store/src/store.rs`、`codex-rs/thread-store/src/lib.rs`、`codex-rs/thread-store/src/local/mod.rs`
- 恢复与闲置卸载：`codex-rs/app-server/src/request_processors/thread_lifecycle.rs`
- 事件与背压：`codex-rs/app-server/src/outgoing_message.rs`
- 插件变更通知：`codex-rs/app-server/src/effective_plugin_change.rs`

持久化部分只审计了上述文件，没有覆盖全部持久化模式及迁移路径。

### 暂不照搬的部分

- 模型上下文、Agent 子会话、沙箱和工具审批。
- Codex 特有的 Thread 业务模型。
- JSONL 历史与 SQLite 索引的完整兼容策略。Atrium 需要结构化状态，但应根据自身查询、迁移、备份和并发需求设计存储。

## OpenCode v2

2026-09-22 新增 `opencode/` 作为第二个只读参考样本，当前基线为 tag `v2.0.12`，提交 `2670273ff1`。参考目标仍是程序架构，不是把 OpenCode 的编码 Agent 功能引入 Atrium。

### 值得参考的机制

- **分层依赖边界**：`schema` 定义领域形状和事件，`protocol` 定义 HTTP API，`core` 承载运行时和持久化，`server` 负责服务组合，`client` 从协议生成调用面，`sdk` 再组合客户端、核心和服务。依赖方向由仓库规则明确约束，避免桌面端或客户端直接依赖核心实现。
- **同一服务的多种宿主**：Node 服务进程监听 HTTP；`fetch` 入口构造同一套 API 供嵌入式运行时或测试宿主使用；CLI 可以连接已存在的后台服务，也可以启动独立的 stdio 服务。服务发现通过注册文件、健康检查、版本和认证信息完成。
- **桌面端作为客户端壳**：Electron 主进程和 preload/IPC 负责窗口、文件、系统能力和本地 UI 状态；业务数据通过生成的 HTTP 客户端访问本地 sidecar 或远程服务。桌面端因此可以同时连接本地、WSL、SSH 或其他服务器。
- **结构化事件和投影**：Schema 区分 durable 与 ephemeral 事件；Core Bus 按 aggregate 维护顺序和序列号，在数据库事务中执行投影，并可保留事件日志。运行时查询使用投影表，事件日志用于恢复、追踪和可靠重放。
- **状态事实与运行状态分离**：OpenCode 将持久化 Session、待处理 inbox、进程内执行协调器和客户端订阅分别管理。写前执行 claim 用于进程异常后的恢复；它不被当作“当前进程仍在运行”的证明。
- **作用域化运行时**：全局服务、Location 作用域服务和 Session 操作分层；LocationServiceMap 根据工作目录或工作空间复用和释放资源。这种结构可以映射到 Atrium 的全局、工作空间和任务/工作流实例作用域。
- **协议生成**：HTTP API 从类型化的协议定义生成 OpenAPI、Promise 客户端和 Effect 客户端，服务端和不同客户端共享同一份契约。
- **插件生命周期和可重算扩展**：插件通过受控上下文注册工具、Hook 或状态变换；插件启用、禁用和配置变化可以触发可重算的派生状态，而不是让插件直接修改数据库内部结构。
- **事件流隔离和背压**：公共事件使用 SSE；每个连接有独立的有限队列，慢客户端超限时只断开自己，不阻塞其他客户端和核心发布者。

### 对 Atrium 的具体启发

- Atrium 可以借鉴 `schema → protocol → core → server → client → desktop/CLI` 的边界，但把领域对象换成 Workspace、Project、Task、Workflow 和 Run。
- “服务”应是核心能力的宿主方式之一：可以由 CLI 启动后台服务，也可以由桌面端嵌入或拉起；不需要把服务、CLI 和桌面端当成三套业务实现。
- 结构化状态应区分事实记录、当前投影和实时通知。任务状态变更应通过明确的命令或领域操作产生，桌面端通过投影和事件更新，而不是让 Agent 直接改 Markdown。
- 工作空间可以成为主要运行时作用域；工作流执行实例可以拥有独立的生命周期、事件流和恢复记录，同时不必采用 OpenCode 的 Session 业务模型。
- 工作流插件可以声明能力、命令、状态变换和事件处理边界。插件的变更应可卸载、可重算、可检查版本，避免插件对核心状态做不可逆的任意写入。
- CLI、Skill、桌面端和未来其他客户端应共享协议生成的接口；Skill 只是一个调用适配层，不应成为唯一的状态一致性保障。

### 不应照搬的部分

- OpenCode 的模型、Provider、Agent、工具调用、Session inbox、上下文压缩和编码任务语义属于其产品业务。
- Bun、Effect、SQLite 以及具体的 Electron 实现是技术选择，不自动成为 Atrium 的技术决策。
- OpenCode 的高复杂度事件溯源、恢复和插件目录机制只有在 Atrium 的并发工作流、长任务恢复或扩展需求确实出现时才应逐步引入。

## Pi

2026-09-22 新增 `pi/` 作为第三个只读参考样本，当前基线为 `main` 提交 `95fbc0499`。Pi 的参考重点需要拆开看：`pi-agent-core` 是 Agent runtime，`chord` 是独立的应用组合运行时，`pi-durable` 是会话、任务和文档持久化设计，`pi-protocol`、`pi-server` 和 `pi-client` 是实验性的远程服务边界。

### 值得参考的机制

- **Agent runtime 与宿主分离**：`Agent` 可以作为进程内运行时被宿主调用，负责模型请求、工具执行、转向输入、事件流和上下文变换；CLI/TUI、服务或其他应用负责宿主体验和资源装配。Atrium 如果未来接入 Agent，可以把它作为可选执行器，而不是把 Agent 逻辑写进状态核心。
- **Chord 的应用组合模型**：插件声明自己提供和依赖的服务，宿主在激活前校验依赖图，并按依赖顺序启动、按逆序释放；Facet 可以把同一个插件拆成 worker、presentation 或其他运行环境中的部分。
- **稳定服务句柄和远程绑定**：Chord 用类型化服务 token 区分 singleton 与 keyed 实例，消费者持有稳定 facade，提供者可以替换或断开；远程边界只规定调用、订阅、快照和更新的语义，不把具体传输方式写死。
- **不可变复制状态**：Chord 的 replicated state 通过事务内 copy-on-write 产生完整新版本，再计算 JSON delta；每个订阅有独立的状态编码器，断开或替换后必须重新 hydrate。这个机制适合桌面端观察工作空间和任务投影。
- **协议路由与展示连接分离**：Pi 协议用 `serverId`、`sessionId` 和 `attachmentId` 组成路由目标；一个持久 Session 可以有多个 presentation attachment。服务端负责路由和连接生命周期，业务服务由应用提供，Session/Harness 本身不跨进程传输。
- **传输边界的严格校验**：Pi 使用版本握手、长度前缀 CBOR、请求取消、相关 ID、错误码和最大帧大小；外层协议只验证严格 JSON 和路由，不替业务服务解析不透明 payload。
- **Pico5 的原子持久化模型**：规范提出一次 Session commit 同时提交不可变 entries、task records 和 Chord documents；只有提交后的状态可观察，外部副作用不能在事务内执行，监听器在提交后异步运行。Task 是可恢复的持久状态机，Document 是带快照和增量操作的结构化 JSON 状态。

### 对 Atrium 的具体启发

- Atrium 可以把“核心状态运行时”和“Agent 执行器”分成两个可替换层：工作空间、任务和工作流状态不依赖某个 Agent runtime。
- 工作流插件可以借鉴 Facet 思路，把声明、核心执行、桌面展示和远程适配拆成不同能力面；宿主负责依赖检查、版本、启动、停用和资源释放。
- 桌面端不必持有工作流对象本身，只需通过稳定服务句柄或协议订阅工作空间/任务投影，并在断线后重新获取完整快照，再应用增量更新。
- 工作流命令可以采用“请求先持久化、执行后异步推进”的输入模型，类似 Pico5 的 InputHandle；这样创建任务、取消任务和等待结果的语义可以由程序保证，而不是依赖 Skill 维护 Markdown。
- 每次工作流状态提交应尽量原子地更新任务状态、运行记录和相关结构化文档；通知在提交后发布，避免桌面端看到尚未落盘的进度。
- Atrium 的协议需要在 Pi 的路由边界之上增加领域 schema 校验。Pi 的 opaque payload 设计有利于通用路由，但不能单独保证 Workspace、Task 和 Workflow 操作正确。

### 需要谨慎的部分

- `pi-durable` 的 Pico5 文档是规范和设计方向，README 明确说明完整 Pico5 尚未实现；不能把它当作已经验证的现成持久化框架。
- `pi-server` 和 `pi-protocol` 当前标注为 experimental；连接认证、服务进程生命周期和业务权限仍由应用负责。
- Chord 的 Facet、远程服务、增量编码和热替换能力很强，但会显著增加实现复杂度。Atrium 首期可以先保留稳定服务接口和完整快照，等插件或多客户端需求明确后再引入增量复制。
- Agent 的转向队列、工具并行、上下文压缩和会话树属于 Agent 产品语义，不能直接成为 Atrium 的任务模型。
