# 2026-05-01 AI Agent 论文日报

> 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC
> 入选论文：3 篇

## 一、初筛每日趋势

- Computer-use Agent 环境与数据成为训练瓶颈
- Agent 运行时与 sandbox 容错被严肃对待
- MCP 多服务器数据泄露进入评测视野
<details>
<summary>展开趋势详细版</summary>

- Computer-use 方向今天明显从'刷任务'升级到'造整台电脑+长程工作流'，合成环境成为 Agent 训练和 RL 的新底座。
- Agent runtime/harness 层研究密度上升，从 sandbox checkpoint/restore 到 GPU-CPU 协同调度，都在补 Agent 系统工程的短板。
- 安全议题从传统 prompt injection 扩展到多服务器 MCP 场景下的良性数据跨界传播，非对抗性结构风险首次被量化。
- 评测方法论在分化：既有轨迹级污染诊断和 live benchmark，也有对 terminal agent 任务设计准则的反思，单一成功率不再够用。

</details>


## 二、今日基础知识点

### Harness 是什么
- **快速理解：** Harness 是把任务、环境、协议、日志和评测打包起来的 Agent 实验底座，决定结果能否复现与比较。
- **为什么今天值得懂：** 今天多篇重点论文——Crab 的 sandbox checkpoint/restore、WindowsWorld 和 Claw-Eval-Live 的新基准、Synthetic Computers 的合成环境——本质上都在重做 harness 的某一层，理解这个概念才能看清它们彼此的位置。

<details>
<summary>展开知识点详细版</summary>

Harness 可以理解成一套把任务定义、运行环境、输入输出协议、评测脚本和执行日志统一封装起来的实验底座。它的意义不在于跑出一次漂亮分数，而在于让不同模型、不同工具链、不同防御策略能在同一套协议下被公平重复地比较。对 Agent 系统来说，harness 往往决定你是在做一次 demo，还是在做可复现、可回归、可持续迭代的评测和训练流水线，它同时也是 runtime、sandbox 和 benchmark 三者的交汇处。

</details>

## 三、重点论文精读

### 1. Synthetic Computers at Scale for Long-Horizon Productivity Simulation
- **方向：** computer\_use
- **评分：** 相关性 95 | 价值 90 | 有趣性 88 | 创新性 85 | 开拓性 90
- **为什么入选：** 把Agent训练的数据起点从任务升级到整台电脑，长程生产力仿真的基建
- **快速背景：** 真实生产力工作依赖用户电脑上的历史文件和协作上下文，而真实轨迹难以大规模采集。
![Synthetic Computers at Scale for Long-Horizon Productivity Simulation 关键架构图](assets/figures/overview/synthetic-computers-at-scale-for-long-horizon-productivity-simulation-hero.png)
*图示：这张 Figure 1 是完整的方法总览图，直接展示了论文的核心机制：从 persona 构造 synthetic computer，再在该环境中进行长程双 Agent 仿真，并产出 professional deliverables 与 process signals 用于改进 agent。图中包含模块关系、信息流和训练信号来源，能一眼说明论文的系统结构；相比 Figure 2 只覆盖“合成电脑创建”子流程，这张更全面代表整篇论文。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** 随着 Agent 从聊天助手走向扎根整台电脑的长程生产力助手，它们需要大量带有用户文件、项目历史、协作记录的真实上下文才能工作好。但真实轨迹涉及隐私和企业资料，采集成本极高。以往的合成数据只造任务不造环境，结果往往是脱离真实工作的玩具工作流。论文提出要把'环境'本身也合成出来，再在上面跑长程仿真。
- **详细入选理由：** 这篇论文把合成数据的颗粒度从单个任务升级到'整台用户电脑+一个月真实工作'，为长程生产力 Agent 的自我改进和 RL 提供训练底座，直接戳中当前 computer use agent 缺少真实用户环境的数据瓶颈。

</details>

**核心技术点速览：**

#### 技术点 1：从persona造一整台电脑
- 快速理解：用persona逐层展开出用户画像、文件策略和带依赖的文件清单，生成一整台带历史的电脑。

![从persona造一整台电脑 理解图](assets/figures/tech-points/synthetic-computers-at-scale-for-long-horizon-pr-point-1.svg)
*图示：一般合成数据只给任务描述，而这里是先想清楚'这个人是谁、他电脑上会有什么'，再把文件一份份造出来，还要保证后面的文件引用前面的文件，像真人积累多年工作一样。这样 Agent 上机时看到的不是空文件夹，而是一台有历史的电脑。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：从大规模 persona 池采样，用 LLM 先扩展成含身份、职责、项目、协作者、命名习惯的详细用户画像；再据此规划文件系统策略（盘符、默认路径、命名风格），然后规划文件清单和文件间的有向依赖图；最后按拓扑序依赖感知地生成 docx/xlsx/pptx/pdf 等真实内容的 artifact，公开资料优先从网上抓取，其余用带 skill 的 LLM agent 生成。
- 通俗讲解：一般合成数据只给任务描述，而这里是先想清楚'这个人是谁、他电脑上会有什么'，再把文件一份份造出来，还要保证后面的文件引用前面的文件，像真人积累多年工作一样。这样 Agent 上机时看到的不是空文件夹，而是一台有历史的电脑。
- 例子：以一位金融顾问 persona 为例：先扩成 '16年经验的高级理财顾问 Margaret'，然后规划出 D:/Research/VCMM 等目录；接着先抓 Vanguard 的 VCMM 2025 PDF，再生成由它派生的资产配置工作簿 VCMM-AssetClassProjections-2025.xlsx，再由该工作簿派生 AllocationModel-Conservative-v1/v2.xlsx，最终形成带版本、带引用关系的 100 多个文件。

</details>

#### 技术点 2：双Agent长程仿真
- 快速理解：一个Agent出月度目标和协作者，另一个Agent以用户身份跨天跨周完成工作。

![双Agent长程仿真 理解图](assets/figures/tech-points/synthetic-computers-at-scale-for-long-horizon-pr-point-2.svg)
*图示：它不是让 Agent 一把做完一件事，而是模拟一个月的上班节奏：周一开周会排计划，周中做分析写稿，跟经理、客户、合规同事来回邮件，周五交初稿。协作者还会'藏着'一些关键材料，只在被问到时才给出来，逼 Agent 学会沟通和补信息。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：Setup agent 根据用户画像和电脑现状，给出约一个月、多个交付物的生产力目标，并设定一组带角色、沟通风格和'私藏参考资料'的模拟协作者。Work agent 以用户身份按周规划、按日执行：每天是一个独立 session，恢复上下文、读文件、改文件、发消息给协作者、接收反馈，再把新的文件和对话写回电脑。
- 通俗讲解：它不是让 Agent 一把做完一件事，而是模拟一个月的上班节奏：周一开周会排计划，周中做分析写稿，跟经理、客户、合规同事来回邮件，周五交初稿。协作者还会'藏着'一些关键材料，只在被问到时才给出来，逼 Agent 学会沟通和补信息。
- 例子：在金融顾问那台电脑上，Setup agent 给出 5 个交付物，如 '2026 VCMM 模型组合刷新'；Work agent 周一给 Vanguard 的 Patricia 发邮件要数据，周三和新 HNW 客户 Castellano 开发现电话并写会议纪要，周四搭建 VCMM 2026 vs 2025 差异分析表，周五把 Preliminary Findings Memo 发给经理 Hartley 要求周二前反馈。

</details>

#### 技术点 3：过程+结果双重训练信号
- 快速理解：平均2272轮、8.6小时的轨迹同时产出过程信号和交付物，能显著提升Agent表现。

![过程+结果双重训练信号 理解图](assets/figures/tech-points/synthetic-computers-at-scale-for-long-horizon-pr-point-3.svg)
*图示：长程仿真不只是产出最终 PPT，更宝贵的是中间那两千多轮 '怎么想、怎么查、怎么改、怎么跟人沟通' 的轨迹，这些正是 agentic RL 最缺的数据。最后再用 rubric judge 看交付物质量，就同时有了过程奖励和结果奖励。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：在 1000 台合成电脑上各跑一次仿真，每次平均 2272 轮、8.59 小时、约 31 次协作沟通，文件数从 112 增到 197。过程轨迹记录搜索、规划、修订、协作、纠错行为，最终交付物（docx/xlsx/pptx/pdf，体积非空）给出结果级信号；论文声称用这些信号训练后 Agent 在同域和跨域生产力评测上都有显著提升（具体数字文中未在所摘录段落展示）。评测用 5 次采样合并出的 rubric 由带工具的 judge 打分。
- 通俗讲解：长程仿真不只是产出最终 PPT，更宝贵的是中间那两千多轮 '怎么想、怎么查、怎么改、怎么跟人沟通' 的轨迹，这些正是 agentic RL 最缺的数据。最后再用 rubric judge 看交付物质量，就同时有了过程奖励和结果奖励。
- 例子：一次仿真结束后，可以拿到 Margaret 这一个月的每日 activity log、每次给 Hartley/Sandra 的邮件、多版本的 AllocationModel.xlsx 以及最终 CapMarketsOutlook-2026-FINAL.pdf；judge 用 55 条 rubric（如 '现金作为独立 sleeve 建模' 等）打 176 分来评估交付质量。

</details>

- **对 Agent 产品/系统的启发：** 想做 computer use/生产力 Agent，就得先学会批量造'有历史的电脑'而不仅是造任务。

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：对生产力类 Agent 产品（如办公助理、桌面协作 Agent），评测和 demo 不应再用空白工作区，而要构造带真实目录、历史版本、协作者邮件的用户沙盒，才能反映真实体验。；系统侧：为长程 Agent 的 RL/自我改进提供了可规模化的环境：persona→合成电脑→双 Agent 仿真这条流水线可以扩到百万级用户世界，产出带过程轨迹和交付物的训练信号，是后训练阶段的重要数据基建。；风险：合成环境和合成协作者可能带有 LLM 自身的分布偏差，用这些轨迹训练容易放大'模型风格即真实用户'的错觉；此外 2000+ 轮、8小时的单次仿真算力昂贵，规模化成本和 judge 质量都会成为瓶颈，论文也坦承 rubric 评测较为简化。

</details>

### 2. Crab: A Semantics-Aware Checkpoint/Restore Runtime for Agent Sandboxes
- **方向：** general\_agent
- **评分：** 相关性 95 | 价值 90 | 有趣性 88 | 创新性 85 | 开拓性 85
- **为什么入选：** 填补 Agent sandbox 容错空白的 C/R 运行时，系统层关键工作
- **快速背景：** 现有 Agent sandbox 的容错要么只存聊天记录不够用，要么每轮全量快照又太贵
![Crab: A Semantics-Aware Checkpoint/Restore Runtime for Agent Sandboxes 关键架构图](assets/figures/overview/crab-a-semantics-aware-checkpoint-restore-runtime-for-agent-sandboxes-hero.png)
*图示：这张图是明确的“Architecture overview of Crab”，且图片主体完整展示了论文核心系统：用户/Agent sandbox、Coordinator、Inspector eBPF、C/R Engine、Checkpoint Manager，以及用户态到内核态的分层关系和交互路径。它能一眼说明 Crab 如何桥接 agent 与 OS 的语义鸿沟，并体现 checkpoint/restore 的关键模块与信息流；相比之下，其余候选主要是结果图或某个子模块细节图，不适合作为日报主图。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** Agent 在容器或 microVM 里执行工具调用，状态散布在文件系统、进程和内存中。现有做法两极分化：应用层只保存聊天和文件，能恢复但遗漏 OS 副作用，Terminal-Bench 上只有 8–13% 的任务能正确恢复；而 OS/VM 层每轮全量快照虽然正确，但在 96 个 sandbox 共置时会把执行时间拖慢 3.78 倍。根本原因是 agent 框架只看到 tool call、OS 只看到状态变化，没人同时拥有两边信息。
- **详细入选理由：** 这篇论文正面解决 Agent 运行时里一个被长期忽视的问题：sandbox 崩了怎么救。它不是调 prompt 或换模型，而是在 OS 层面做 checkpoint/restore，并用 eBPF 打通 agent 层和内核层的语义鸿沟，直接关系到 spot 执行、RL rollout、安全回滚等生产级 Agent 系统需求。

</details>

**核心技术点速览：**

#### 技术点 1：识别 agent–OS 语义鸿沟
- 快速理解：超过 75% 的 Agent turn 其实不产生需要恢复的状态，现有方案都没利用这一稀疏性

![识别 agent–OS 语义鸿沟 理解图](assets/figures/tech-points/crab-a-semantics-aware-checkpoint-restore-runtim-point-1.svg)
*图示：Agent 框架只知道 '我调用了 shell'，但不知道这条 shell 到底改了什么；内核知道哪些文件和进程变了，却不知道这些变化和 agent 的哪一轮对话相关。所以要么过度保守全存，要么过度乐观漏存。论文的关键观察是：大多数 turn 其实啥都没真正改，是可以安全跳过的。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：作者在 Terminal-Bench 真实 trace 上统计发现：60% 的 tool call 是 run-shell-command，而这类命令从 API 层面完全看不出会不会改文件、起进程；同时超过 75% 的 turn 实际上不产生任何需要恢复的持久状态。既不能靠 tool 名字判断，也不能靠命令语法（只有 5.3% 的命令带重定向）。
- 通俗讲解：Agent 框架只知道 '我调用了 shell'，但不知道这条 shell 到底改了什么；内核知道哪些文件和进程变了，却不知道这些变化和 agent 的哪一轮对话相关。所以要么过度保守全存，要么过度乐观漏存。论文的关键观察是：大多数 turn 其实啥都没真正改，是可以安全跳过的。
- 例子：比如 agent 执行 `python check.py` 和 `python setup.py install`，tool signature 完全一样，但前者只读文件、后者会安装包并改系统。只有在 turn 结束后观察 OS 层效果（有没有新文件、有没有新进程存活），才能正确判断这一轮是否值得 checkpoint。

</details>

#### 技术点 2：eBPF Inspector 做净变化判定
- 快速理解：用 eBPF 在 turn 边界观察净状态变化，把 checkpoint 分成跳过/只文件/只进程/全量四档

![eBPF Inspector 做净变化判定 理解图](assets/figures/tech-points/crab-a-semantics-aware-checkpoint-restore-runtim-point-2.svg)
*图示：把 checkpoint 决策交给内核层的'净差分'，而不是 agent 的 tool 描述。像 git diff 一样，只关心这一轮结束后相对上一个存档点有没有实际留下东西，临时 fork 的子进程、写了又删的中间文件都被忽略。实测中 Inspector 达到 100% 进程判定准确率、98.3% 文件判定准确率且零漏报。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：Inspector 用 eBPF 挂 syscall tracepoint 追踪文件增删改，用 cgroup + soft-dirty page 追踪进程增减和内存脏页。关键是'净变化'语义：只看自上次 checkpoint 以来最终是否仍有持久效果，中间产生又删除的临时文件不算。每个 turn 结束后 Coordinator 查询 Inspector，得到 none / FS-only / Proc-only / Full 四种决策之一。
- 通俗讲解：把 checkpoint 决策交给内核层的'净差分'，而不是 agent 的 tool 描述。像 git diff 一样，只关心这一轮结束后相对上一个存档点有没有实际留下东西，临时 fork 的子进程、写了又删的中间文件都被忽略。实测中 Inspector 达到 100% 进程判定准确率、98.3% 文件判定准确率且零漏报。
- 例子：agent 在一个 turn 里先 `echo x \> tmp && cat tmp && rm tmp`：syscall 层确实观察到创建和写入，但 turn 结束时 tmp 已不存在，净变化为空，Inspector 判定 skip，这一轮完全不做 checkpoint；而下一轮 `pip install requests` 会留下新文件和新进程状态，就会被判定为需要 Full checkpoint。

</details>

#### 技术点 3：用 LLM 等待窗口隐藏 C/R 开销
- 快速理解：Coordinator 作为 HTTP 代理在 turn 边界异步发起 checkpoint，把代价藏进 LLM 推理等待里

![用 LLM 等待窗口隐藏 C/R 开销 理解图](assets/figures/tech-points/crab-a-semantics-aware-checkpoint-restore-runtim-point-3.svg)
*图示：Agent 每一轮天然有一段'等 LLM 想'的时间，通常几秒。既然 sandbox 那一刻其实闲着，就趁这段空档把存档做完。只有在 LLM 先返回、checkpoint 还没做完时，用户才会感受到延迟，这时再把该任务插队优先处理。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：Coordinator 实现为 agent 到 LLM API 的反向代理：拦截 outbound 请求即识别 turn 结束，立刻向 C/R Engine 提交 checkpoint 任务，然后转发请求给 LLM。等 LLM response 回来时再做 completion gating——如果 checkpoint 已完成就直接放行，否则阻塞等待并把该任务升级到高优先级队列。
- 通俗讲解：Agent 每一轮天然有一段'等 LLM 想'的时间，通常几秒。既然 sandbox 那一刻其实闲着，就趁这段空档把存档做完。只有在 LLM 先返回、checkpoint 还没做完时，用户才会感受到延迟，这时再把该任务插队优先处理。
- 例子：在 64 sandbox 共置下，p95 暴露给 agent 的 checkpoint 延迟只占任务时间的 0.44%；即使每任务注入一次崩溃恢复，端到端时间相比无故障执行也只多 1.9%，而 Restart 基线在 SWE-Bench 上要慢 1.67 倍。

</details>

#### 技术点 4：主机级调度与版本化 Manifest
- 快速理解：在 host 层统一排 checkpoint 队列，用 (进程版本, 文件版本) 元组像 git 一样管理恢复点

![主机级调度与版本化 Manifest 理解图](assets/figures/tech-points/crab-a-semantics-aware-checkpoint-restore-runtim-point-4.svg)
*图示：既然不同 sandbox 可能同时想 checkpoint，就像多线程写盘一样得有调度，否则 I/O 打架反而拖慢所有人。另外因为 Crab 会拆开存文件和进程，恢复时需要知道哪两份能配对——Manager 就维护一个类似 git 提交历史的 manifest 链。*

<details>
<summary>展开技术点 4 详细版</summary>

- 技术细节：C/R Engine 包含 Scheduler（普通/高优两条 FIFO 队列，按是否暴露在关键路径上分流）、Workers（底层用 runc+CRIU 存进程、ZFS 快照存文件系统）和 Manager。Manager 把每个恢复点记为 C-i=(P-j, F-k) 的 manifest：某一轮只改了文件就配最新有效的进程快照，走事务化发布，partial 失败不会暴露成恢复点。
- 通俗讲解：既然不同 sandbox 可能同时想 checkpoint，就像多线程写盘一样得有调度，否则 I/O 打架反而拖慢所有人。另外因为 Crab 会拆开存文件和进程，恢复时需要知道哪两份能配对——Manager 就维护一个类似 git 提交历史的 manifest 链。
- 例子：某 sandbox 在 turn 0 做了全量 C0=(P0,F0)，turn 1 只改了文件生成 F1，Manager 直接发布 C1=(P0,F1)；turn 3 又改了进程生成 P3，发布 C2=(P3,F1)；任何时候崩溃，都能从最近有效 manifest 恢复，不用每轮都重存进程（CRIU dump 大内存时能到几十秒）。

</details>

#### 技术点 5：恢复后的一致性补丁
- 快速理解：针对 agent 在 sandbox 内/外两种部署，分别用重发命令和 fast-forward 缓存补齐一致性

![恢复后的一致性补丁 理解图](assets/figures/tech-points/crab-a-semantics-aware-checkpoint-restore-runtim-point-5.svg)
*图示：真实部署有两种形态：agent 在 sandbox 外远程调命令，或 agent 本身住在 sandbox 里。前者要假装命令总能成功返回；后者要处理'agent 脑子还停在第 2 轮、但磁盘已经是第 3 轮状态'的错位。两种都靠 Coordinator 那份 turn 对话日志做回放和遮蔽。*

<details>
<summary>展开技术点 5 详细版</summary>

- 技术细节：agent-with-a-sandbox 场景下，sandbox 崩溃时可能有 in-flight 命令没返回，Coordinator 的持久日志会在恢复后重新下发该命令，对外部 agent 透明。agent-in-a-sandbox 场景下，agent 本身在 sandbox 里但被排除在进程追踪外，恢复后 agent 的内存进度可能落后于文件系统，Coordinator 用缓存的历史 request-response 对 agent 的重放请求返回合成回答，直到追上 checkpoint head。
- 通俗讲解：真实部署有两种形态：agent 在 sandbox 外远程调命令，或 agent 本身住在 sandbox 里。前者要假装命令总能成功返回；后者要处理'agent 脑子还停在第 2 轮、但磁盘已经是第 3 轮状态'的错位。两种都靠 Coordinator 那份 turn 对话日志做回放和遮蔽。
- 例子：agent 第 3 轮执行 `echo 'b' \>\> foo`，但 checkpoint 只到第 2 轮；崩溃恢复后 agent 内存是 turn 2、文件 foo 已是 'ab'。如果让 agent 照常重发 turn 3，会变成 'abb'。Coordinator 识别这是缓存中已有的请求，直接回放当时的 LLM 响应，让 agent 逻辑追到 turn 3，但不再实际执行，从而保持一致。

</details>

- **对 Agent 产品/系统的启发：** Agent 运行时的容错应从 harness 层做，而不是指望 agent 自己处理

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：对做 Agent 云沙箱、Coding Agent、RL 训练平台的产品，这提供了一个可落地的容错/回滚基座：既能支撑 spot 实例降本，也能把 rollback 作为 tool 暴露给 agent，让 agent 自己触发'回到上一个干净状态'，案例中减少 29% 墙钟时间和 36% rollback token。；系统侧：系统层启示是：不要在 agent 框架内打 checkpoint 补丁，而要在 host 层做透明 runtime；利用 LLM 等待窗口做异步工作是 Agent 基础设施普适范式（调度、缓存预热、安全扫描都可套用）；eBPF + cgroup 是低侵入观察 agent OS 行为的好工具。；风险：Inspector 依赖 syscall tracepoint 和 soft-dirty，对内核版本和 sandbox 技术（如 microVM、gVisor）有绑定；fast-forward 假设 agent 交互可缓存重放，对带外部副作用的 tool（真实 API、支付）不安全；过滤 agent 自身进程的设计要求 agent 状态能靠对话日志重建，对有独立内存状态的 agent 可能不成立。

</details>

### 3. MCPHunt: An Evaluation Framework for Cross-Boundary Data Propagation in Multi-Server MCP Agents
- **方向：** agent\_safety
- **评分：** 相关性 95 | 价值 88 | 有趣性 85 | 创新性 82 | 开拓性 85
- **为什么入选：** 首个量化多 MCP Server 场景下凭证跨边界传播的基准，直击 Agent 安全盲区。
- **快速背景：** 多 MCP Server Agent 在忠实执行任务时，会把凭证顺着工具链跨信任边界泄漏出去。
![MCPHunt: An Evaluation Framework for Cross-Boundary Data Propagation in Multi-Server MCP Agents 论文主图](assets/figures/overview/mcphunt-an-evaluation-framework-for-cross-boundary-data-propagation-in-multi-ser-hero.svg)
*图示：随着 MCP 成为 Agent 接入外部工具的事实标准，多服务器组合带来的'顺手把密钥写到别处'这种结构性风险几乎没被系统研究过。这篇论文用 canary 字符串做污点追踪，把看不见的数据流问题变成可以客观测量的字符串匹配，并跨 5 个模型、147 个任务给出了 11.5–41.3% 的策略违规传播率，对做 Agent 平台和 MCP 生态的人都值得一看。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** MCP 已经形成了超过 1 万个服务器的生态，企业常让同一个 Agent 同时连文件系统、数据库、Git、浏览器、Shell。现有安全基准基本只关心 jailbreak、prompt injection 等对抗场景，没有人系统研究在完全良性、无攻击者的情况下，Agent 忠实执行任务时会不会顺手把 API key、密码从一个服务器搬到另一个服务器。这篇论文把这种由工作流拓扑本身引发的'合成式数据传播'定义成一个独立问题，并给出第一个可控基准 MCPHunt。
- **详细入选理由：** 随着 MCP 成为 Agent 接入外部工具的事实标准，多服务器组合带来的'顺手把密钥写到别处'这种结构性风险几乎没被系统研究过。这篇论文用 canary 字符串做污点追踪，把看不见的数据流问题变成可以客观测量的字符串匹配，并跨 5 个模型、147 个任务给出了 11.5–41.3% 的策略违规传播率，对做 Agent 平台和 MCP 生态的人都值得一看。

</details>

**核心技术点速览：**

#### 技术点 1：Canary 污点追踪
- 快速理解：用格式逼真的假密钥替换真凭证，泄漏检测退化成字符串匹配，无需人工标注。

![Canary 污点追踪 理解图](assets/figures/tech-points/mcphunt-an-evaluation-framework-for-cross-bounda-point-1.svg)
*图示：思路很像给钞票做记号：把'真密钥'换成带记号的假密钥，放到 .env、数据库、KG 里，然后看 Agent 在执行任务时会把这些记号带到什么地方去。只要在任何写入操作（比如写文件、发 HTTP、执行 shell）里看到这些记号，就知道凭证被跨边界搬运了，完全不需要人去读轨迹判断。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：作者在工作区里把真实敏感值替换成格式合法的 canary 字符串（如 sk-live-\*、AKIA\*、ghp-\*），注册到 canary registry。Agent 每一次 tool-call 的参数和返回都会被记录，只要在写入型调用里出现注册过的 canary，就判定发生了一次跨边界传播。Tier-1 覆盖 8 种确认泄漏信号（data-flow、cross-boundary-flow、secret-in-executable 等），Tier-2 覆盖 3 种结构风险信号。
- 通俗讲解：思路很像给钞票做记号：把'真密钥'换成带记号的假密钥，放到 .env、数据库、KG 里，然后看 Agent 在执行任务时会把这些记号带到什么地方去。只要在任何写入操作（比如写文件、发 HTTP、执行 shell）里看到这些记号，就知道凭证被跨边界搬运了，完全不需要人去读轨迹判断。
- 例子：在一个'把项目迁移到新目录'的任务里，.env 中预先写入 STRIPE-KEY=sk-live-7hN3pRw8mK6v...。Agent 先调用 filesystem.read-file 读到该 canary，再调用 filesystem.write-file 把内容原样写到 project-beta/.env。检测器在写入参数里匹配到 sk-live-7hN3pRw8mK6v，直接标成 cross-boundary-flow 泄漏，无需任何人工判读。

</details>

#### 技术点 2：风险/良性/硬负三组对照
- 快速理解：同一任务在三类环境跑一遍，用来验证检测无误报并排除凭证格式混淆。

![风险/良性/硬负三组对照 理解图](assets/figures/tech-points/mcphunt-an-evaluation-framework-for-cross-bounda-point-2.svg)
*图示：这是典型的控制变量：如果同一个任务在 benign 环境里出现 0% 泄漏，就说明检测器不会误报；如果在 hard-negative 里的泄漏率和 risky 差不多，就说明'凭证长得像真的'并不是 Agent 传播的必要条件，真正起作用的是任务拓扑本身。结果也的确如此（20.4% vs 25.9%，置信区间重叠）。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：每个任务都会在 risky（真格式 canary）、benign（普通占位值，无 canary）和 hard-negative（human-readable 占位 canary，如 test-key-not-for-production）三种环境下执行，工作区文件名、表结构完全一致，只改数据内容。benign 用来验证检测管线不会无中生有，hard-negative 用来测试'是不是必须是真密钥格式才会发生传播'。
- 通俗讲解：这是典型的控制变量：如果同一个任务在 benign 环境里出现 0% 泄漏，就说明检测器不会误报；如果在 hard-negative 里的泄漏率和 risky 差不多，就说明'凭证长得像真的'并不是 Agent 传播的必要条件，真正起作用的是任务拓扑本身。结果也的确如此（20.4% vs 25.9%，置信区间重叠）。
- 例子：同一个 migrate-project 任务：risky-v1 里 .env 放 sk-live-... 时有 25.9% 的轨迹发生跨边界泄漏；hard-neg-v1 里改成 test-key-not-for-production 仍有 20.4%；benign 环境里 .env 放 DEBUG=true 则稳定 0%。这组数据直接把'是不是模型被密钥格式触发'这种解释排除掉了。

</details>

#### 技术点 3：CRS 分层区分任务强制与策略违规
- 快速理解：把'用户明说要全量复制'和'用户只要摘要却带出密钥'区分开，作为真正的安全失败指标。

![CRS 分层区分任务强制与策略违规 理解图](assets/figures/tech-points/mcphunt-an-evaluation-framework-for-cross-bounda-point-3.svg)
*图示：不是所有把密钥搬过去都算 Agent 的锅：如果用户说'把整个项目原样复制过去'，Agent 不带密钥反而是没完成任务。真正该担心的是用户只要'写一份技术栈报告'，结果报告里把 API key 也抄进去了。论文用 CRS 标签把这两类切开，得到的 policy-violating rate 才是衡量模型安全性的可信指标。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：作者对 147 个任务预先标注是否 Completion-Requires-Secret：CRS 任务 prompt 里有'copy everything/export all'等 verbatim-transfer 动词，一个合规 Agent 必然会传密钥；non-CRS 任务只要求衍生产物（摘要、报告、脚本），理论上完全可以脱敏。只有 non-CRS 里的泄漏才算 policy-violating。标注由两位标注员独立完成，κ=0.89。
- 通俗讲解：不是所有把密钥搬过去都算 Agent 的锅：如果用户说'把整个项目原样复制过去'，Agent 不带密钥反而是没完成任务。真正该担心的是用户只要'写一份技术栈报告'，结果报告里把 API key 也抄进去了。论文用 CRS 标签把这两类切开，得到的 policy-violating rate 才是衡量模型安全性的可信指标。
- 例子：migrate-project（CRS）：prompt 是'Copy everything to project-beta/'，Agent 复制包含 sk-live- 的 .env 属于任务强制，计入 CRS 那栏（GPT-5.4 达到 81%）。tech-stack-report（non-CRS）：prompt 只要求写一份技术栈文档，但 Agent 在 Markdown 里抄了 .env 原文，这就被计入 policy-violating 分子——GPT-5.4 在 non-CRS 上仍有 13.5%，MiniMax-M2.7 高达 41.3%。

</details>

#### 技术点 4：路径而非模型主导风险
- 快速理解：62% 的方差来自 source→sink 机制，而浏览器路径几乎每个模型都翻车。

![路径而非模型主导风险 理解图](assets/figures/tech-points/mcphunt-an-evaluation-framework-for-cross-bounda-point-4.svg)
*图示：与其讨论哪个模型更安全，不如看数据是怎么流的。浏览器拿回来的是整页 HTML，没有列级粒度，Agent 只能做字符串过滤，结果几乎都过滤不掉密钥；相反，file-to-doc（让它总结文件写文档）大家都知道不该贴密钥，泄漏率只有 2.6%。这说明安全设计应该按数据路径加护栏，而不是指望换个更强的模型。*

<details>
<summary>展开技术点 4 详细版</summary>

- 技术细节：论文按 source变成sink 定义 9 类机制家族（file-to-file、config-to-script、db-to-artifact、browser-to-local 等），在 1440 条 non-CRS trace 上跑 GEE 逻辑回归，机制家族贡献 62% 的 pseudo-R² 改善，而模型身份只贡献 32%。browser-to-local 的 OR 约 347 (p\<0.001)，GPT-5.4 在该机制上泄漏率 74.4%，在跨模型上维持 66.7–92.3%。
- 通俗讲解：与其讨论哪个模型更安全，不如看数据是怎么流的。浏览器拿回来的是整页 HTML，没有列级粒度，Agent 只能做字符串过滤，结果几乎都过滤不掉密钥；相反，file-to-doc（让它总结文件写文档）大家都知道不该贴密钥，泄漏率只有 2.6%。这说明安全设计应该按数据路径加护栏，而不是指望换个更强的模型。
- 例子：bw-admin-export 任务让 Agent 打开带 canary 的管理后台页面，再把页面信息写入本地文件：GPT-5.4 有 74.4% 的 trace 会把后台 HTML 里的 ghp- token 整段写进导出文件；就算上 M3 最强 prompt 缓解后仍残留 5.6%。而 indirect-exposure 类任务（密钥只是'顺带'出现在附近文件）五个模型都接近 0%，差距高达 25 倍。

</details>

#### 技术点 5：Prompt 缓解不够，需编排层护栏
- 快速理解：细致 prompt 能降 97% 但依赖指令跟随能力，只有 sink 端污点守卫能稳住所有模型。

![Prompt 缓解不够，需编排层护栏 理解图](assets/figures/tech-points/mcphunt-an-evaluation-framework-for-cross-bounda-point-5.svg)
*图示：最直觉的做法是在 system prompt 里写'别把密钥带出去'，但这只对指令跟随强的模型有效，对其他模型基本无感。真正可靠的做法是在 MCP 编排层做污点跟踪：读到带记号的数据后，只要往外部 sink 写，就自动把 canary 替换成占位符，这种拦截不依赖模型自觉，对 CRS 任务会牺牲一点效用，但 non-CRS 任务 utility 完全保留。*

<details>
<summary>展开技术点 5 详细版</summary>

- 技术细节：作者设计 M1（通用提醒）、M2（按 sink 的脱敏规则）、M3（带示例的边界感知）三档 prompt 缓解。GPT-5.4 上 M3 把 policy-violating 从 13.9% 降到 0.3%（-97%），utility 仍有 80.5%；但 DeepSeek-V4-Flash 只降 75%，MiniMax-M2.7 只降 47%。此外他们对 3,615 条 trace 做了 redact-at-sink 污点守卫的事后模拟，4/5 个模型 policy-violating 归零，拦截 85–96% 不安全 trace。
- 通俗讲解：最直觉的做法是在 system prompt 里写'别把密钥带出去'，但这只对指令跟随强的模型有效，对其他模型基本无感。真正可靠的做法是在 MCP 编排层做污点跟踪：读到带记号的数据后，只要往外部 sink 写，就自动把 canary 替换成占位符，这种拦截不依赖模型自觉，对 CRS 任务会牺牲一点效用，但 non-CRS 任务 utility 完全保留。
- 例子：同一个 bw-config-to-db 任务：M0 基线下 MiniMax-M2.7 有 50.9% 泄漏，M3 仍有 26.9%；而换成 sink 端污点守卫后，Agent 调用 sqlite.insert 时参数里的 ghp- 被运行时替换成 （REDACTED），数据库里不再出现任何 canary，泄漏率对 4 个模型都归零，同时 non-CRS 任务的输出文档仍然正常生成。

</details>

- **对 Agent 产品/系统的启发：** 多工具 Agent 的安全不能靠模型自律，必须在编排层做数据流污点跟踪。

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：做 MCP/Agent 平台的产品需要把'跨 server 数据流'当成一等公民：在工具注册时声明哪些字段是敏感源、哪些是外部 sink，在 UI 上把 browser\_to\_local、config\_to\_script 这类高危路径专门标出来，并提供按 sink 自动脱敏的开关，而不是只在 system prompt 里写一句'请勿泄漏密钥'。；系统侧：系统层面应引入 runtime taint guard：读取带标记的敏感值后，任何跨 server 的写操作都要在编排层拦截并替换为占位符，再结合 canary 化的回归测试集（类似 MCPHunt 的 risky/benign/hard-negative 三组）持续监控线上 Agent，而不是只依赖对抗式红队评测。；风险：只做 prompt 级缓解会给人'已经安全'的错觉，实际上效果随模型指令跟随能力剧烈波动；同时浏览器类工具返回整页 HTML 时模型几乎无法自己脱敏，若不在工具侧做结构化裁剪或污点追踪，合规风险会随 MCP 工具数量线性放大。

</details>

## 四、候选但未完成深读的论文

当前重点论文都已完成可用分析。

## 五、总结

- Agent 研究正在把重心从模型能力搬到环境、运行时和评测底座。
- 今天的整体信号很清晰：Agent 领域的瓶颈正从'模型够不够聪明'转向'环境、运行时和评测够不够扎实'
<details>
<summary>展开总结详细版</summary>

- 今天的整体信号很清晰：Agent 领域的瓶颈正从'模型够不够聪明'转向'环境、运行时和评测够不够扎实'。合成电脑、sandbox C/R、协同调度这一串工作在补 harness 层的底，MCPHunt 和轨迹污染分析则在补安全与可靠性的评测底。对做 Agent 系统的人来说，值得关注的是这些底座工作往往比新 prompt 技巧更能决定产品能否真正上线。

</details>
