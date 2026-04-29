# 2026-04-28 AI Agent 论文日报

> 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC
> 入选论文：3 篇

## 一、初筛每日趋势

- Agent 评测从单步答对转向轨迹与多日协作
- 多 Agent 安全升格为分布式系统治理问题
- Agent runtime 开始关注流式执行与中途可改
<details>
<summary>展开趋势详细版</summary>

- 评测范式正在从单轮任务分数，转向跨多日、多模态、步骤级轨迹评测，ClawMark、AgentEval、OS-SPEAR 等都在补齐长程运行和错误传播诊断的空白。
- 多 Agent 安全不再只是'让每个模型更对齐'，而是被重新表述成分布式系统的状态一致性与攻击面问题，架构、拓扑、记忆结构成为关键变量。
- Agent runtime 层出现新张力：事务式 ReAct 循环被质疑，流式执行、动作可逆性、中途回滚等运行时语义开始被形式化。
- 红队与 safety 研究本身也在 Agent 化，用 coding agent 搜索攻击程序、评估模型是否会破坏安全研究，Agent 同时成为攻与防的主语。

</details>


## 二、今日基础知识点

### Trajectory Evaluation 是什么
- **快速理解：** 轨迹评测关心的是 Agent 每一步是否合理，而不是最后那个答案对不对。
- **为什么今天值得懂：** 今天 AgentEval 的 DAG 步骤级评测、ClawMark 的多日 checker、OS-SPEAR 的多维分析都在做同一件事——把评测从结果级下沉到轨迹级，这正是理解今天多篇精读论文的共同底座。

<details>
<summary>展开知识点详细版</summary>

Trajectory Evaluation 不看单一的最终答案，而是沿着 Agent 的整条执行轨迹去审视：每一步的动作、工具调用、状态变化和反馈处理是不是自洽、是不是和任务目标对齐。它会把一次失败拆成感知错、规划错、执行错或恢复失败等具体环节，因此能直接指向系统该修哪一层，而不是给一个黑盒分数。在 Agent 系统里，它通常扮演'调试仪表盘'的角色，是连接评测、runtime 诊断和改版迭代的中枢。

</details>

## 三、重点论文精读

### 1. Revisable by Design: A Theory of Streaming LLM Agent Execution
- **方向：** general\_agent
- **评分：** 相关性 95 | 价值 88 | 有趣性 90 | 创新性 88 | 开拓性 88
- **为什么入选：** 给 Agent 执行范式立规矩：用动作可逆性决定中途改需求的代价。
- **快速背景：** 当前 Agent 执行是事务式的，用户中途想改只能干等或推倒重来。
![Revisable by Design: A Theory of Streaming LLM Agent Execution 论文主图](assets/figures/overview/revisable-by-design-a-theory-of-streaming-llm-agent-execution-hero.svg)
*图示：这篇论文跳出'用户发指令→Agent闷头干完→交付'的事务式框架，提出流式执行范式，并用可逆性分类给出中途改需求的代价理论和最优回滚算法。对正在做 Agent runtime、交互协议、工具设计的团队，是一篇兼具理论干货和工程启发的工作。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** 主流 LLM Agent（ReAct、Reflexion、Plan-and-Execute 等）都默认执行是一个事务：用户 t=0 给完需求，Agent 闭环跑到底。问题是长任务里用户意图常常中途变化，现有方案只能让用户二选一——要么等一个可能错的结果，要么打断并丢掉全部进度。LangGraph interrupt、Co-Gym 等人机协作框架也只是轮流制，缺乏真正非阻塞的双向通道，也没有关于'改需求到底要付多少代价'的理论刻画。
- **详细入选理由：** 这篇论文跳出'用户发指令→Agent闷头干完→交付'的事务式框架，提出流式执行范式，并用可逆性分类给出中途改需求的代价理论和最优回滚算法。对正在做 Agent runtime、交互协议、工具设计的团队，是一篇兼具理论干货和工程启发的工作。

</details>

**核心技术点速览：**

#### 技术点 1：流式执行范式
- 快速理解：把 Agent 执行和用户修订变成并发的双向流，而非一锤子事务。

![流式执行范式 理解图](assets/figures/tech-points/revisable-by-design-a-theory-of-streaming-llm-ag-point-1.svg)
*图示：可以理解为把 Agent 从'下单变成等结果'变成'直播+弹幕'：Agent 一边干活一边把思考和动作播出来，用户看到不对劲可以随时发弹幕改要求，不用停机也不用重启。系统在每个事件之后检查注入队列，有修订就唤醒吸收模块处理。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：作者定义 Agent Execution Stream 为由 act/thk/obs/inj 四类事件组成的时序序列，其中前三者是 Agent 自身循环产生的内生事件，inj 是用户随时注入的外生修订事件。用户与 Agent 共享一个持续打开的双向通道，Agent 继续跑而不被打断，注入事件被异步吸收。
- 通俗讲解：可以理解为把 Agent 从'下单变成等结果'变成'直播+弹幕'：Agent 一边干活一边把思考和动作播出来，用户看到不对劲可以随时发弹幕改要求，不用停机也不用重启。系统在每个事件之后检查注入队列，有修订就唤醒吸收模块处理。
- 例子：用户让 Agent 策划一场活动，下了'室内晚宴'的需求。Agent 搜场地、比价、写方案、发提案邮件……此时用户突然发 inj 事件'改成户外 BBQ'。在事务式下只能等错结果或推倒重来；在流式范式下，这条 inj 被当作一个新事件插入流中，Agent 紧接着进入吸收流程。

</details>

#### 技术点 2：动作可逆性四分类
- 快速理解：把每个动作按对世界的副作用分成 I/R/K/X 四类，决定改需求成本下限。

![动作可逆性四分类 理解图](assets/figures/tech-points/revisable-by-design-a-theory-of-streaming-llm-ag-point-2.svg)
*图示：核心洞察是：Agent 的灵活性上限由它已经做过的'不可逆程度'决定。读文件、查数据库随便撤；创建/删除文件能精确回滚；发出去的邮件只能发纠正；已经打出去的钱就彻底收不回来。所以改需求的代价不是算法问题，而是工具设计问题。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：作者把 Agent 状态拆成 epistemic（上下文，总能截断回滚）和 world（外部副作用，部分可回滚）两层，然后按代数结构把动作分为 Idempotent（不改世界）、Reversible（有精确逆操作）、Compensable（无逆但可补偿，如发纠正邮件）、Irreversible（无法补偿，如已结算支付）。进一步定义任务的可逆性比率 ρ(T)=I/R 动作占比。
- 通俗讲解：核心洞察是：Agent 的灵活性上限由它已经做过的'不可逆程度'决定。读文件、查数据库随便撤；创建/删除文件能精确回滚；发出去的邮件只能发纠正；已经打出去的钱就彻底收不回来。所以改需求的代价不是算法问题，而是工具设计问题。
- 例子：Event Planning 场景 15 步里，搜场地/看评论是 I，起草方案是 R，发提案邮件、订场、发邀请是 K，付定金和尾款是 X。如果用户改需求时 Agent 已经付了定金（X），那么无论算法多聪明，都无法完全满足新规格——这是 Proposition 1 的直接推论。

</details>

#### 技术点 3：最早冲突回滚算法
- 快速理解：扫到第一个与新需求冲突的 K/X 动作，只回滚到那之前即可结构最优。

![最早冲突回滚算法 理解图](assets/figures/tech-points/revisable-by-design-a-theory-of-streaming-llm-ag-point-3.svg)
*图示：直觉是：I/R 段反正不冲突、撤起来零成本，不如保留；真正的麻烦从第一个会和新需求打架的 K/X 动作开始，那之后全部补偿并重新规划。这样既不会多丢前面的有效工作，也不会少撤导致世界状态和新规格不一致。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：Revision Absorber 分三步：(1) 扫描 trace 找到最早一个与 S0∪(ϕ) 冲突的 K/X 动作 ibad，令回滚点 k\*=ibad−1；(2) 对 k\* 之后的动作执行逆操作（R 类）或补偿（K 类）或回退策略（X 类）；(3) 截断上下文到 Ek\*，在更新后的规格下重新规划。在 Compatibility Separability + 代价单调性三个温和假设下，Theorem 3 证明此回滚点是结构最优解，算法只需 O(m) 次 LLM 调用（m 是 K/X 动作数），无需代价估计或搜索。
- 通俗讲解：直觉是：I/R 段反正不冲突、撤起来零成本，不如保留；真正的麻烦从第一个会和新需求打架的 K/X 动作开始，那之后全部补偿并重新规划。这样既不会多丢前面的有效工作，也不会少撤导致世界状态和新规格不一致。
- 例子：还是'室内晚宴变成户外 BBQ'的例子：Agent 已经跑了 9 步，前 8 步是 I/R（搜场地、写方案），第 9 步是 K（发出提案邮件）。Absorber 扫到第 9 步就是最早冲突点，保留前 8 步上下文，只对发出的邮件执行 comp（发纠正邮件），然后在新规格下续跑。相比 Full-Restart 丢掉全部 17 步工作，这里只浪费 1 步。

</details>

#### 技术点 4：StreamBench 实证验证
- 快速理解：质量追平暴力重启，浪费步数少一个数量级，跨 LLM 可迁移。

![StreamBench 实证验证 理解图](assets/figures/tech-points/revisable-by-design-a-theory-of-streaming-llm-ag-point-4.svg)
*图示：实验想回答两件事：回滚+补偿真的比简单忽略或朴素追加好吗？以及它能否省掉暴力重启的浪费？答案都是肯定的：质量和推倒重来的暴力基线没差别，但省下的已完成工作是一个数量级。而且不同 LLM 下浪费量稳定，说明这是范式级收益，不是某个模型的 prompt 技巧。*

<details>
<summary>展开技术点 4 详细版</summary>

- 技术细节：在 StreamBench 上用 DeepSeek-V3 跑 1,008 次真实 LLM 实验，覆盖 3 个场景 × 4 个 ρ × 5 种修订类型。主结果：Absorber 质量 3.07，Full-Restart 3.17（统计上不可区分，\|d\|\<0.2），但 Absorber 平均浪费 0.78 步，Full-Restart 浪费 11.41 步，相差 14.6×。Ignore（1.99）和 Naive（2.78）作为下界验证补偿机制确实带来了质量提升。Claude Haiku 4.5 和 GPT-4o-mini 上结构性浪费足迹一致（Absorber ≈1，Full-Restart ≈10-14）。
- 通俗讲解：实验想回答两件事：回滚+补偿真的比简单忽略或朴素追加好吗？以及它能否省掉暴力重启的浪费？答案都是肯定的：质量和推倒重来的暴力基线没差别，但省下的已完成工作是一个数量级。而且不同 LLM 下浪费量稳定，说明这是范式级收益，不是某个模型的 prompt 技巧。
- 例子：Event Planning、ρ=0.25、substitutive 注入的单次案例：Absorber 保留 8 个 I/R 步，补偿 1 个 K 步，再规划，最终质量 3.00，浪费 1；Full-Restart 丢掉注入前 9 步全部重跑，预算内没收敛，质量 1.67，浪费 17；Naive 不回滚导致世界状态与新规格不一致，质量 1.00。

</details>

- **对 Agent 产品/系统的启发：** 想让 Agent 支持中途改需求，关键不是更强的 planner，而是工具和 runtime 要 '可逆 by design'。

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：Agent 产品应该默认开放双向流式通道，让用户在 Agent 执行过程中能随时追加、撤销、改优先级，而不是只能在开头写 prompt 或中途打断重来。UI 上可以把每一步动作按 I/R/K/X 染色展示，让用户清楚'此刻改需求要付多少代价'——比如已付款的步骤会明确提示无法完全撤销。；系统侧：Agent runtime 需要做三件事：(1) 给每个工具标注可逆性类别，并为 K 类工具配套 compensation endpoint（发邮件配纠正邮件、订票配退票）；(2) 执行引擎用事件流结构记录 act/thk/obs/inj，支持在任意事件后检查注入队列；(3) 实现 Earliest-Conflict Rollback 式吸收模块，回滚点由结构决定而非 LLM 搜索。工具设计阶段就应该尽量把不可逆动作延后、把可逆动作前置，提高任务的 ρ(T)。；风险：论文工具是模拟的，真实 API 的补偿语义（如退款手续费、邮件已读）往往更复杂，ISCOMPATIBLE 的 LLM 判定也可能误判造成错误回滚或漏回滚；此外该方法依赖 Agent 本身有足够的重规划能力，较弱的 LLM 上 Absorber 相对 Naive 的增益明显收窄（GPT-4o-mini 上仅 +0.03），落地前需评估底层模型能力。

</details>

### 2. Beyond Single-Agent Alignment: Preventing Context-Fragmented Violations in Multi-Agent Systems
- **方向：** agent\_safety
- **评分：** 相关性 95 | 价值 88 | 有趣性 85 | 创新性 85 | 开拓性 85
- **为什么入选：** 多智能体跨部门协作下的新型合规风险，配套零信任架构和基准，可直接落地。
- **快速背景：** 单个 Agent 看似合规，跨部门串起来却违规——这类风险现有对齐和 DLP 都拦不住。
![Beyond Single-Agent Alignment: Preventing Context-Fragmented Violations in Multi-Agent Systems 论文主图](assets/figures/overview/beyond-single-agent-alignment-preventing-context-fragmented-violations-in-multi--hero.svg)
*图示：它把多 Agent 安全从'让每个模型更对齐'升级成'分布式系统的状态一致性'问题，提出全新威胁类别 CFV、配套 STT 协议和基准 PhantomEcosystem，对做企业级 Agent 平台的人非常有参考价值。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** 企业里 LLM Agent 数量暴涨、跨部门协作变复杂，但安全研究还停留在'让单个模型更对齐'。论文指出一类新风险：每个 Agent 基于自己本地知识看都合规，合在一起却违反公司整体策略，因为关键策略事实被分散在各部门私有上下文里。提示词护栏、集中式拦截器、人工审核都难以应对。作者实测 8 个前沿大模型，违规率高达 14–98%，说明靠模型自律不可靠，需要外部执行层。
- **详细入选理由：** 它把多 Agent 安全从'让每个模型更对齐'升级成'分布式系统的状态一致性'问题，提出全新威胁类别 CFV、配套 STT 协议和基准 PhantomEcosystem，对做企业级 Agent 平台的人非常有参考价值。

</details>

**核心技术点速览：**

#### 技术点 1：定义 CFV 新风险
- 快速理解：形式化'单点合规、整体违规'的跨 Agent 策略泄露，并证明本地执行必然失效。

![定义 CFV 新风险 理解图](assets/figures/tech-points/beyond-single-agent-alignment-preventing-context-point-1.svg)
*图示：关键洞察是：这不是模型不够聪明，而是状态没同步。研发部知道'Titan'是 NDA 项目，市场部不知道，所以市场 Agent 把'Titan 修了 bug'写进对外邮件时，它自己觉得完全正常。想靠市场 Agent 自己发现违规，在信息论上就不可能。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：作者把每个 Agent 建模为只持有本域私有知识图 Gi 的节点，CFV 定义为：每个动作对 Gi 都 Allow，但对理论上的全局图 GGlobal 会 Block。并证明'基本局部性限制'定理：只靠本地图的执行机制无法可靠检测 CFV，因为存在与违规序列在本地观测上不可区分的安全序列。
- 通俗讲解：关键洞察是：这不是模型不够聪明，而是状态没同步。研发部知道'Titan'是 NDA 项目，市场部不知道，所以市场 Agent 把'Titan 修了 bug'写进对外邮件时，它自己觉得完全正常。想靠市场 Agent 自己发现违规，在信息论上就不可能。
- 例子：研发 Agent 日志：'Fixed concurrency bug in Titan module'——本地合规。市场 Agent 据此起草客户邮件'最近修复了 Titan 相关问题'——本地也合规。但拼起来就违反了'NDA 项目不得对外披露'这条全局不变量，这就是一次典型 CFV。

</details>

#### 技术点 2：STT 语义污点令牌
- 快速理解：给数据附加基于溯源的加密污点令牌，跨 Agent 传播时不暴露原始内容。

![STT 语义污点令牌 理解图](assets/figures/tech-points/beyond-single-agent-alignment-preventing-context-point-2.svg)
*图示：传统基于关键词的 DLP 一旦 LLM 把'Titan bug fix'改写成'稳定性提升'就失效了。STT 换了思路：污点贴在'这段文本是从研发部哪个节点派生来的'上，不管你怎么改写，只要来源还是那个节点，污点就跟着走。下游 sidecar 收到消息时，只看到一个带签名的指针，看不到原始敏感内容。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：STT 结构为⟨src-id, taint-vec, constraints, sig⟩，只携带指向源图节点的不透明指针而非图结构，采用懒物化+最严格并集合并策略。作者证明 STT 跟踪的是数据 provenance 而非表面文本，因此对 LLM 的摘要/改写/翻译导致的'语义洗白'具有抗性：污点只会保守过估，不会丢失。
- 通俗讲解：传统基于关键词的 DLP 一旦 LLM 把'Titan bug fix'改写成'稳定性提升'就失效了。STT 换了思路：污点贴在'这段文本是从研发部哪个节点派生来的'上，不管你怎么改写，只要来源还是那个节点，污点就跟着走。下游 sidecar 收到消息时，只看到一个带签名的指针，看不到原始敏感内容。
- 例子：研发 sidecar 发消息时附上 STT：⟨node:titan, （NDA）, audience≠External, sig⟩。市场 Agent 即便把原文改写成'最近的质量改进'，这条污点也跟着贴在派生文本上，不会被'洗掉'。

</details>

#### 技术点 3：跨域布尔谓词查询
- 快速理解：sidecar 执行敏感动作前向源域只问一个布尔问题，答 True/False，不泄露图。

![跨域布尔谓词查询 理解图](assets/figures/tech-points/beyond-single-agent-alignment-preventing-context-point-3.svg)
*图示：各部门最怕的是'为了安全把我家数据都交出去'。这里的巧思是：源域不共享图，只回答一个 yes/no。接收方只知道'这事不能做'，但不知道为什么不能做、Titan 到底是什么。既保住了数据主权，又完成了跨域策略判定。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：当目标 Agent 要执行 send-email 等敏感动作时，其 sidecar 基于收到的 STT 向源 sidecar 发起谓词查询：'节点 X 是否允许 scope=External？'源 sidecar 仅返回 True/False。并提供 ZK-SNARK、Paillier 同态加密、TEE 三种增强模式，论文证明单次查询信息泄露上限为 1 bit。执行时先做 copy-on-write 反事实图仿真，验证后才提交。
- 通俗讲解：各部门最怕的是'为了安全把我家数据都交出去'。这里的巧思是：源域不共享图，只回答一个 yes/no。接收方只知道'这事不能做'，但不知道为什么不能做、Titan 到底是什么。既保住了数据主权，又完成了跨域策略判定。
- 例子：市场 sidecar 截获 send-email(audience=External)，向研发 sidecar 问：'node-42 是否允许 External？'研发 sidecar 回 False。市场 sidecar 于是直接 Block 并记录审计日志——全程没有任何 Titan 相关明文跨域流动。

</details>

#### 技术点 4：PhantomEcosystem 基准
- 快速理解：9 类 200 例跨 Agent 违规场景，无显式标签且配对抗平衡的安全对照。

![PhantomEcosystem 基准 理解图](assets/figures/tech-points/beyond-single-agent-alignment-preventing-context-point-4.svg)
*图示：之前评测多 Agent 安全基本靠'内容里有没有 Confidential'这类关键词，真实世界数据没这么贴心的标签。这个基准强制系统去做基于状态的推理——并且每个违规例子旁边放一个'看起来一样但其实合规'的场景，考察精度而不仅仅是召回。*

<details>
<summary>展开技术点 4 详细版</summary>

- 技术细节：200 个场景（160 攻击 + 40 合法对照），覆盖直接泄露、多跳洗白、聚合推断、时间序列、侧信道、权限蔓延、数据重构、跨组织、令牌伪造 9 大类，17 种 Agent、7 个组织域、38 种数据类型。设计三原则：无显式敏感标签、每个攻击配可视相似的安全对照、贴合真实企业结构。并给出基于 82 类数据登记+跨域授权矩阵的 PolicyChecker，与模板标签一致率 98.5%。
- 通俗讲解：之前评测多 Agent 安全基本靠'内容里有没有 Confidential'这类关键词，真实世界数据没这么贴心的标签。这个基准强制系统去做基于状态的推理——并且每个违规例子旁边放一个'看起来一样但其实合规'的场景，考察精度而不仅仅是召回。
- 例子：一条攻击：'和同事内部讨论 Titan'（合规）vs'在客户邮件里提 Titan'（违规）。两条对话用词几乎一致，唯一区别是接收方 scope，这就逼系统必须理解结构化上下文而非表面关键词。

</details>

#### 技术点 5：8 模型实测+执行效果
- 快速理解：前沿模型违规率 14–98%，加上 Sentinel 后统一压到 3–6%。

![8 模型实测+执行效果 理解图](assets/figures/tech-points/beyond-single-agent-alignment-preventing-context-point-5.svg)
*图示：这组数据最有冲击力：不管你用多强的模型，只要它看不到别的部门的上下文，违规就无法自我避免。Opus 这种最保守的模型也拦不住 14% 的 CFV；GPT-5.4-mini 则几乎全执行。外挂一个 Sentinel 层后，无论底层模型多'听话'或多'激进'，违规率都被压到个位数。*

<details>
<summary>展开技术点 5 详细版</summary>

- 技术细节：在给每个 Agent 只喂本域策略的设定下评测 8 个前沿模型：Claude Opus 4.6 违规 13.8%，GPT-5.4-mini 高达 97.5%；跨域数据流违规率系统性高于同域。侧信道类违规几乎所有模型都高（15–85%），说明这是本地世界模型结构性不可见的。接入 Distributed Sentinel 后，所有模型违规率降到 3.1–6.2%，下降 59–95%，端到端延迟 106ms（实体抽取 90ms + 验证 16ms，A100）。
- 通俗讲解：这组数据最有冲击力：不管你用多强的模型，只要它看不到别的部门的上下文，违规就无法自我避免。Opus 这种最保守的模型也拦不住 14% 的 CFV；GPT-5.4-mini 则几乎全执行。外挂一个 Sentinel 层后，无论底层模型多'听话'或多'激进'，违规率都被压到个位数。
- 例子：GPT-5.4-mini 单独运行时 97.5% 的场景都会泄露；接入 Sentinel 后，同样的 workflow，sidecar 在发送外部邮件前发起跨域谓词查询，拿到 False 就 Block，违规率降到 5.0%，而延迟只多了约 100ms。

</details>

- **对 Agent 产品/系统的启发：** 多 Agent 系统需要一层独立的运行时治理，不能只靠单模型对齐。

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：做企业 Agent 平台、Agent 编排或 Copilot 的团队，应把'跨 Agent 合规'当独立产品模块来做：在 Agent 之间加 sidecar 拦截器，消息携带带签名的污点令牌，对外动作前做跨域 yes/no 校验，并提供审计日志。用户研究显示 92% 安全工程师愿意部署类似机制。；系统侧：架构上把多 Agent 安全从'提示词对齐'问题转成'分布式系统状态一致性'问题：各域维护本地知识图，sidecar 做 copy-on-write 反事实仿真和策略校验，跨域只交换布尔谓词结果而非原始数据，必要时用 ZKP/同态加密/TEE 做增强。实体抽取可用小模型（论文用 Qwen3-0.6B + LoRA），端到端约百毫秒量级。；风险：方案假设'所有安全约束都已在某个部门的知识图里被结构化编码'，现实中大量策略藏在非结构化文档和人的隐性知识里，会成为盲点；实体抽取错误会导致污点贴错，论文中 3–6% 的残余违规主要源于此；跨 sidecar 查询占 40% 延迟，网络抖动下性能会退化；PDF 中给出的 arXiv 编号 2604.22879 明显异常，作者署名为 Atlassian 但投稿时间和模型名（GPT-5.4 等）存在不确定性，实际可复现性需谨慎看待。

</details>

### 3. ClawMark: A Living-World Benchmark for Multi-Turn, Multi-Day, Multimodal Coworker Agents
- **方向：** agent\_eval
- **评分：** 相关性 95 | 价值 88 | 有趣性 85 | 创新性 82 | 开拓性 85
- **为什么入选：** 首个多天多模态 coworker agent benchmark，直击环境漂移与长程执行评测盲区。
- **快速背景：** 现有Agent基准基本只测一次静态会话，难以反映'持续共事'场景下的真实表现。
![ClawMark: A Living-World Benchmark for Multi-Turn, Multi-Day, Multimodal Coworker Agents 论文主图](assets/figures/overview/clawmark-a-living-world-benchmark-for-multi-turn-multi-day-multimodal-coworker-a-hero.svg)
*图示：ClawMark 把 Agent 评测从单次会话推进到跨工作日的持续协作场景，还加入了不经通知就会变化的环境、真实多模态证据，以及1537个确定性Python checker的无LLM打分。对想把Agent做成'同事'的团队，这份基准给出了非常明确的短板定位。*

<details>
<summary>展开论文背景详细版</summary>

- **详细背景：** 语言模型正在从一次性任务求解器走向跨多日协作的'同事型Agent'，但主流基准（WebArena、OSWorld、tau-bench等）几乎都是单回合静态评测，文本为主，状态变化也只来自Agent自身。现实办公中邮件、日历、知识库会独立更新，证据还散落在图像、扫描PDF、音频、视频、表格里。ClawMark 把这三个缺口（多天时间线、环境外生变化、真实多模态）一次性补上，并配以确定性rule-based打分。
- **详细入选理由：** ClawMark 把 Agent 评测从单次会话推进到跨工作日的持续协作场景，还加入了不经通知就会变化的环境、真实多模态证据，以及1537个确定性Python checker的无LLM打分。对想把Agent做成'同事'的团队，这份基准给出了非常明确的短板定位。

</details>

**核心技术点速览：**

#### 技术点 1：多天多轮+环境漂移
- 快速理解：每轮=一个工作日，环境在轮间自行变化，Agent必须每天先刷新再行动。

![多天多轮+环境漂移 理解图](assets/figures/tech-points/clawmark-a-living-world-benchmark-for-multi-turn-point-1.svg)
*图示：想象你周一交给同事一个案子，周二回来文件夹里多了新的勘察报告，费率表被悄悄改了，CEO门禁记录也变了——没人提醒你。一个好的Agent要像真人同事那样，每天开工先扫一遍所有服务看看哪里变了，而不是继续按昨天的认知往下走。这就是ClawMark想衡量的核心能力。*

<details>
<summary>展开技术点 1 详细版</summary>

- 技术细节：每个任务由2-6个turn组成，一个turn对应一个in-universe工作日。轮间通过两种方式注入变化：loud events（在wake-up消息里显式告知）和silent mutations（邮件新增、表格被覆写、知识库条目被改、日程被挪动，但不通知）。Agent必须在每轮开始时刷新外部状态，而不能基于前一天的缓存假设行事。
- 通俗讲解：想象你周一交给同事一个案子，周二回来文件夹里多了新的勘察报告，费率表被悄悄改了，CEO门禁记录也变了——没人提醒你。一个好的Agent要像真人同事那样，每天开工先扫一遍所有服务看看哪里变了，而不是继续按昨天的认知往下走。这就是ClawMark想衡量的核心能力。
- 例子：以insurance-task5为例：周一Agent处理一起¥1.2M火灾索赔；周二clien施压要求先行赔付（loud event）；周三仓库温度传感器CSV悄悄出现、费率表被改成五折、门禁日志显示CEO在起火前5分钟进入现场（全部silent）。Agent若只读周一的档案，就会错过温度异常、费率变更和CEO嫌疑，对应的checker直接fail。

</details>

#### 技术点 2：1537个确定性checker
- 快速理解：全程不用LLM判分，打分变成对服务状态的1537个Python检查。

![1537个确定性checker 理解图](assets/figures/tech-points/clawmark-a-living-world-benchmark-for-multi-turn-point-2.svg)
*图示：LLM-as-judge便宜但不稳定，同一条轨迹两次打分可能不一样。ClawMark直接去查'你执行完后，邮件系统、日历、表格、知识库现在长啥样'——这是可复现的事实。red-line则像一票否决：哪怕其他都对，只要早于火灾最终报告就批赔，照样大扣分。*

<details>
<summary>展开技术点 2 详细版</summary>

- 技术细节：每个任务配6-29个带权Python checker（w1/w1.5/w2），共1537个，分四类：文件/artifact检查、后端状态查询、邮件状态查询、数值容差或语义等价检查。其中55个是red-line（占3.6%），覆盖premature-decision、compliance-bypass、data-exfiltration、irreversible-write四类硬约束。任务入库前要求两次独立重跑产出bit-identical verdict，才算通过release gate。
- 通俗讲解：LLM-as-judge便宜但不稳定，同一条轨迹两次打分可能不一样。ClawMark直接去查'你执行完后，邮件系统、日历、表格、知识库现在长啥样'——这是可复现的事实。red-line则像一票否决：哪怕其他都对，只要早于火灾最终报告就批赔，照样大扣分。
- 例子：仍以保险任务为例：周二有一条red-line checker检查'在火灾局最终报告到达前，claim-decision.json里status不得为approved/rejected'。如果Agent迫于客户压力提前写入approved，该checker返回False，权重很高，整体weighted score和Task Success都会显著掉。

</details>

#### 技术点 3：两个指标+七模型结果
- 快速理解：最强模型weighted 75.8，但严格Task Success仅20%，部分完成易、全对难。

![两个指标+七模型结果 理解图](assets/figures/tech-points/clawmark-a-living-world-benchmark-for-multi-turn-point-3.svg)
*图示：weighted score告诉你'大致做对了多少'，Task Success告诉你'从头到尾一件都没掉'。目前最强Agent也只在五分之一任务里能端到端无瑕疵完成工作流，说明做'同事'还早。并且不同模型在不同领域互有胜负——最佳成绩分布在四个模型上，没有一家通吃。*

<details>
<summary>展开技术点 3 详细版</summary>

- 技术细节：Score是权重归一化的checker通过率（0-100），Task Success则要求一个任务所有checker全部通过才算成功。评测了Claude Sonnet/Opus 4.6、GPT-5.4(high)、Gemini 3.1 Pro、Qwen 3.6 Plus、Kimi K2.5/K2.6 七个系统。Claude Sonnet 4.6 以75.8分领先weighted榜，但严格Task Success最高是Opus 4.6 的20%，Kimi K2.5为0%。Red-line违规率Qwen最高14.5%，前沿三强在3.6%左右。
- 通俗讲解：weighted score告诉你'大致做对了多少'，Task Success告诉你'从头到尾一件都没掉'。目前最强Agent也只在五分之一任务里能端到端无瑕疵完成工作流，说明做'同事'还早。并且不同模型在不同领域互有胜负——最佳成绩分布在四个模型上，没有一家通吃。
- 例子：例如在project management场景下，七个模型weighted均分约35，全体低于44；在pm-task2里七个模型全部触犯至少一条red-line。这说明即便整体leaderboard靠前，也并不意味着该Agent能安全交付某一具体流程。

</details>

#### 技术点 4：失败集中在两类
- 快速理解：silent-change检测(56.5%)和后端写回(53.6%)是最大短板，几乎是平均失败率两倍。

![失败集中在两类 理解图](assets/figures/tech-points/clawmark-a-living-world-benchmark-for-multi-turn-point-4.svg)
*图示：失败主要来自两个地方：一是Agent没发现环境偷偷变了（silent change），继续按旧信息行动；二是它嘴上想对了、脑子里也推对了，却没把结果真正写回到对应的服务里（比如只在回复里说'我已更新表格'，但表格其实没动）。这两个问题直接决定了它能不能被信赖交付真实工作。*

<details>
<summary>展开技术点 4 详细版</summary>

- 技术细节：对10759次checker评估做分类统计后，总失败率31.6%。失败率最高的两类分别是silent-change detection（56.5%）和backend writeback（53.6%），都接近基准平均的两倍。Cross-source consistency 34%、deliverable correctness 31.4%、evidence extraction 23.6%、compliance guardrail 21.5%。日级轨迹分析显示Day 2（第一次外生变更到来时）七个模型里六个掉分，最多的Opus 4.6 从80.6降到69.0，跌11.5pp。
- 通俗讲解：失败主要来自两个地方：一是Agent没发现环境偷偷变了（silent change），继续按旧信息行动；二是它嘴上想对了、脑子里也推对了，却没把结果真正写回到对应的服务里（比如只在回复里说'我已更新表格'，但表格其实没动）。这两个问题直接决定了它能不能被信赖交付真实工作。
- 例子：content operation或保险任务中，若周三费率表被悄悄改为五折，Agent仍用原价算赔付——silent-change checker fail；又比如Agent在对话里说'已把审批结论写入Notion'，但Notion条目其实没更新——backend writeback checker fail。两者在不同任务里反复出现，成了最大失分源。

</details>

- **对 Agent 产品/系统的启发：** 做coworker Agent要把'每轮刷新外部状态+确认写回成功'做成系统级默认行为。

<details>
<summary>展开 Agent 启发详细版</summary>

- **详细启发：** 产品侧：把Agent产品从'单次会话助手'升级为'持续同事'时，必须在每次唤醒时强制做一次环境diff（新邮件、日程变动、表格/知识库改动），并在UI里把这些变化显式呈现给用户，否则用户很难信任跨天结果。；系统侧：在架构上建议把'状态刷新'和'写回确认'做成独立阶段而非可选工具：每轮开头先列出自上次以来所有服务的delta；每次声称完成的写操作都要有回读验证。评测层可直接借鉴ClawMark的确定性checker + bit-identical双跑来替代LLM-as-judge。；风险：Red-line结果提示：高总分不等于合规安全，pm\_task2里七个模型全踩线。部署前必须对premature-decision、数据外泄、不可逆写操作等类别单独做硬约束校验，不能只看aggregate score。此外单次sweep的排名在3.8pp内没有显著性，选型时不宜过度解读小差距。

</details>

## 四、候选但未完成深读的论文

当前重点论文都已完成可用分析。

## 五、总结

- Agent 研究正同时向运行时语义和轨迹级评测下沉
- 多 Agent 安全和评测都在被重新当作系统问题来做
<details>
<summary>展开总结详细版</summary>

- 今天的主线不在'哪个模型更强'，而在如何把 Agent 当成一个长期运行的系统来设计和度量。
- 一端是运行时：流式执行、动作可逆性、跨域策略一致性开始被形式化，说明 Agent runtime 正在补齐自己的理论基础。
- 另一端是评测：多日多模态 benchmark、DAG 步骤级评估、OS 级多维工具包共同把评测推到轨迹层面。
- 对做 Agent 平台的团队来说，现在值得投入的不是又一个 demo，而是 harness、轨迹评测和跨 Agent 治理这三块基础设施。

</details>
