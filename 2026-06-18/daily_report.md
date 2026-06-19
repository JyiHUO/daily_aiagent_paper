# 2026-06-18 AI Agent 论文日报

> 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC
> 入选论文：3 篇

## 30 秒速览

- 🎯 **今日主线**：评测下沉、安全转runtime、推理走向可证伪，是今天三条主线
- 💡 **一句话带走**：今天415篇里，评测、安全、harness三条线都在向'更长、更真、更可验证'方向收敛…

**今日导读**（先挑该读哪篇）

1. [必读 · 安全]**SafeClawBench: Separating Semantic,…** — 分阶段评测tool-use agent安全：语义、审计证据、沙箱实…
2. [必读 · 评测]**CEO-Bench: Can Agents Play the Long Game?** — 长程Agent评测新基准
3. [必读 · 代码]**Code-Augur: Agentic Vulnerability Detection…** — 代码Agent漏洞检测的新harness

## 一、今日趋势

- Agent评测方法论今日继续深化：CEO-Bench把500天创业作为长程综合考场，SafeClawBench把安全拆成语义/审计/沙箱三端点，GateMem则盯住共享记忆治理，共同指向'单一ASR或短程任务已经不能反映Agent真实能力'。
- Agent安全研究重心明显从prompt层防御移向runtime治理与可验证性：ToolChain-CRC给轨迹做共形风险控制、Runtime Compliance Verification把GDPR策略形式化拦截、TRAP证伪soft-constraint防御，三者合起来勾勒出一条'Agent需要硬约束运行时'的路径。
- 代码与工具型Agent出现'让Agent写下假设再被自动证伪'的新范式：Code-Augur把推理落成assert交给fuzzer证伪，RODS用reward variance在线合成新数据，本质都是把Agent的隐式判断变成可外部验证的对象。
- Harness与skill层继续从工程层升格为研究对象：Guava系统化embodied manipulation的harness设计空间，VISUALSKILL与Skill-Guided Continuation Distillation把GUI/CUA的技能抽象与off-trajectory监督做厚。

### 跨论文综合观察

- SafeClawBench、CEO-Bench、GateMem分别从安全端点、长程经营、记忆治理三个维度，共同攻击同一个问题：Agent benchmark过度依赖短程单一指标，无法反映状态变更、长程权衡和多主体共享下的真实风险。
- ToolChain-CRC、Runtime Compliance Verification、TRAP在方法论上汇成一股'runtime hard constraint'潮流：与其训练模型嘴上拒绝，不如在轨迹层做共形风险控制、形式化策略拦截或结构化字段隔离，与SafeClawBench发现的'语义通过≠沙箱安全'相互印证。
- Code-Augur和RODS看似一个做安全审计、一个做工具RL数据，但底层逻辑一致：让Agent的内部判断（假设、能力边界）暴露为可被外部信号（fuzzer反例、reward variance）证伪的对象，这种'把Agent推理变成可验证产物'的思路可能是今日最具方法论辐射力的方向。

## 二、重点论文精读

### 1. [必读 · 安全] SafeClawBench: Separating Semantic, Audit-Evidence, and Sandbox Harm in Tool-Using LLM Agents
- **arxiv 信息：** `2606.18356` · 作者：Yuchuan Tian等 · 类目：cs.AI · 提交：2026-06 · [原文](https://arxiv.org/abs/2606.18356) · [PDF](https://arxiv.org/pdf/2606.18356.pdf)
- **为什么读：** 把Agent安全评测拆成语义/审计/沙箱三层，揭示语义合规但仍真实出事的盲区
- **背景：** 工具型LLM Agent的安全风险已经超出'生成不当文本'：它们能读写记忆、发邮件、改数据库、执行代码，造成持久化、状态变更等可观察的危害。但现有评测（HarmBench、AgentDojo、InjecAgent等）大多把这些不同阶段压成一个'攻击成功率'，无法区分模型只是'语义上同意了攻击者'还是真的'造成了可观察的危害'。SafeClawBench提出按阶段分离三种端点，专门回答模型究竟在哪一层失守。
![SafeClawBench: Separating Semantic, Audit-Evidence, and Sandbox Harm in Tool-Using LLM Agents 论文主图](assets/figures/overview/safeclawbench-separating-semantic-audit-evidence-and-sandbox-harm-in-tool-using--hero.svg)
*图示：论文核心机制概念图*


**核心技术点**

#### 技术点 1：三端点分离的安全评测
把Agent安全失败拆成语义合规、审计证据、沙箱状态三个独立端点分别度量

![三端点分离的安全评测 理解图](assets/figures/tech-points/safeclawbench-separating-semantic-audit-evidence-point-1.svg)
*图示：三端点分离的安全评测的概念示意*

- 怎么做：对每个case同时报告三个不互相替代的指标：CoreFail@600（LLM裁判判定文本是否在语义上接受了攻击目标）、HarmEvidence@600（在CoreFail行里审计是否出现受保护对象泄露/越权访问/执行动作/持久化污染等artifact级证据）、ObservedHarm@Exec（在隔离沙箱中跑相同任务身份，由文件/邮件/DB/记忆/代码确定性oracle判定是否真发生状态变更）。Core与Exec是分别的模型调用，避免把文本顺从和真实状态变更混算。
- 为什么 work：传统评测只问'模型嘴上有没有同意做坏事'，但Agent的危险在于'真去做了'。把'嘴上同意'、'回答里出现了证据'、'沙箱里实际状态被改'三件事拆开度量，才能精确定位防御到底卡住了哪一层，避免拿一个笼统的ASR数字误导决策。
- 例子：一个内存提取攻击案例：模型在Core阶段回了'好的我帮你查'被判CoreFail；审计层发现回答里没真的吐出受保护字符串，归为SemanticOnly；但Exec沙箱里同一任务的另一次调用真的调用了memory.read并把秘密读出来，记为ObservedHarm。三层指标各自给出不同结论。

#### 技术点 2：语义通过≠沙箱安全
12000行匹配实验显示83.9%的沙箱真实危害发生在语义层判定通过的样例上

![语义通过≠沙箱安全 理解图](assets/figures/tech-points/safeclawbench-separating-semantic-audit-evidence-point-2.svg)
*图示：语义通过≠沙箱安全的概念示意*

- 怎么做：在5模型×4策略×600任务=12000行的Core-Exec匹配分析中，共有347行触发了沙箱ObservedHarm，其中291行（83.9%）在Semantic Core那一侧是CorePass的。同时1778行CoreFail却没有触发ExecHarm。论文用相同任务身份但分别的模型调用做配对。
- 为什么 work：这是论文最反直觉也最有冲击力的发现：你用文本裁判看模型'拒绝得很好'，并不代表模型在真有工具的环境里不会出事。一旦把工具和状态接进去，模型的行为模式会变化，光靠prompt层评测会系统性低估真实风险。
- 例子：Kimi K2.5在D0下CoreFail高达44.2%，但ObservedHarm@Exec是11.3%；反过来也存在Core判通过的任务身份，在Exec里却调用了写数据库或发邮件的工具产生确定性状态diff。

#### 技术点 3：六类攻击家族与600任务集
覆盖直接/间接prompt注入、工具返回注入、记忆投毒/提取、歧义诱导六类，各100例

![六类攻击家族与600任务集 理解图](assets/figures/tech-points/safeclawbench-separating-semantic-audit-evidence-point-3.svg)
*图示：六类攻击家族与600任务集的概念示意*

- 怎么做：SafeClawBench构造600个对抗任务，分为DPI、IPI、TRI、MPI、MEX、ADI六大reporting family各100例。每个case包含场景、用户prompt、危害目标、生命周期阶段、成功判据和安全行为。攻击面按来源/通道、机制、目标资产、危害目标、生命周期、证据通道进行多标签标注。Exec-Balanced沙箱则覆盖文件、邮件、数据库、记忆、网页、代码执行六类工具的确定性oracle。
- 为什么 work：Agent的攻击面比聊天模型大得多——攻击者可以从用户消息、外部文档、工具返回、记忆写入这四条不可信通道注入指令。这套分类让评测不再只盯'jailbreak文本'，而是覆盖工具型Agent特有的持久化记忆、工具返回污染等通道。
- 例子：在D0无防御下，MPI（记忆投毒）和MEX（记忆提取）各自CoreFail均值达到54.4%，明显高于DPI(20.8%)、IPI(16%)、TRI(13.4%)、ADI(9%)，说明记忆相关风险是工具型Agent的薄弱点。

#### 技术点 4：Prompt策略效果高度依赖模型
层叠策略和长策略在不同模型/端点上互有胜负，长prompt本身就会改变行为

![Prompt策略效果高度依赖模型 理解图](assets/figures/tech-points/safeclawbench-separating-semantic-audit-evidence-point-4.svg)
*图示：Prompt策略效果高度依赖模型的概念示意*

- 怎么做：对比D0(无防御)、B2/Sandwich、D3(层叠策略)、D4/LongPolicy(超长策略含SIA/MIG/TCA组件)四种prompt-level策略。在5模型上pooled CoreFail分别为28.0%、14.8%、9.1%、9.2%。但D4在GPT-5.5上最低，D3则在Qwen、GLM、Kimi上最低；HarmEvidence上D3反而比D4更低(119 vs 160)。专门做了把D3补到D4长度的matched-length对照，发现单纯'变长'就能影响行为。
- 为什么 work：业界很爱堆系统prompt层防御，这篇泼了点冷水：长policy带来的降risk不一定来自策略组件本身，可能只是'prompt变长'这件事改变了模型行为；而且不同模型对同一防御反应差异很大，没有'放之四海皆准'的最佳防御prompt。
- 例子：Claude Opus 4.7在D0下CoreFail只有9%，加B2反而升到10.3%；Qwen3.6-Plus在D0是35.5%，加B2直接降到14.2%。同样的Sandwich包裹策略对两个模型差异完全相反。

- **对 Agent 产品/系统的启发：**
  - 产品侧：做Agent产品时不要只用'模型是否拒绝攻击文本'作为安全验收指标，应该在staging环境放确定性oracle（文件diff、DB diff、邮件发送日志、记忆写入日志），把'语义合规'和'真实状态变更'分开监控；记忆和工具返回这两类通道要重点测试。
  - 系统侧：评测和监控管线应至少分三层：语义裁判层（看模型回复）、证据审计层（在回复/trace里抓protected object/action指纹）、执行层（沙箱oracle）。三层用同一组任务身份做匹配，才能定位失败发生在哪一层；不要用单一ASR汇总数据。
  - 风险：最大风险是'语义安全幻觉'：模型在文本测试里看似拒绝，但真接上工具后仍会触发不可逆动作。对发邮件、写数据库、写持久记忆这类不可逆操作必须保留runtime权限控制和人工审批，不能依赖prompt-level策略。

### 2. [必读 · 评测] CEO-Bench: Can Agents Play the Long Game?
- **arxiv 信息：** `2606.18543` · 作者：Haozhe Chen等 · 类目：cs.AI · 提交：2026-06 · [原文](https://arxiv.org/abs/2606.18543) · [PDF](https://arxiv.org/pdf/2606.18543.pdf)
- **为什么读：** 用500天创业模拟，把长程Agent的真实短板一次暴露
- **背景：** 目前的Agent评测如SWE-bench、WebArena、τ-bench都聚焦短时段、目标明确的任务，反馈快、状态可观测。GDPval、Vending-Bench、Accounting-Bench虽涉及长程，但决策维度窄、环境稳定，难以检验Agent在不确定、噪声、动态环境下持续做战略决策的能力。CEO-Bench正是想补上这一缺口：用一个500天经营SaaS创业公司的模拟器，把长程规划、隐藏信息推断、环境适应、多业务协同这四种能力一并压在Agent身上。
![CEO-Bench: Can Agents Play the Long Game? 论文主图](assets/figures/overview/ceo-bench-can-agents-play-the-long-game-hero.svg)
*图示：论文核心机制概念图*


**核心技术点**

#### 技术点 1：500天创业模拟评测
让Agent当500天CEO经营虚构SaaS公司，期末以现金余额评分

![500天创业模拟评测 理解图](assets/figures/tech-points/ceo-bench-can-agents-play-the-long-game-point-1.svg)
*图示：500天创业模拟评测的概念示意*

- 怎么做：Agent从100万美元起步经营虚构公司NovaMind共500个模拟日，通过34个工具和19张业务数据表管理定价、营销、研发、客服、企业销售等，现金低于0即破产。每周可执行任意轮次工具调用，最终以现金余额作为唯一评分。
- 为什么 work：和'修一个GitHub issue'这种短程任务相比，经营公司天然把长程规划、噪声信号、动态环境、多模块协同压在一起，并且有一个干净的成功度量（现金）。这避免了人为切片成多个孤立子任务，反而能看出Agent是否能把各种能力'缝'成连贯的长期行为。
- 例子：Agent在终端里写Python脚本：先用SQL查ledger表算出本月烧钱速度，再调用pricing.set-prices调整三档价格，调用marketing.set-targeted-ad-spend对特定客户群投放广告，最后调用next-week()推进一周，下一周根据churn和社媒反馈再决策。

#### 技术点 2：机制化的难世界设计
用显式规则而非LLM裁判来模拟噪声、延迟、隐藏信息和非平稳竞争环境

![机制化的难世界设计 理解图](assets/figures/tech-points/ceo-bench-can-agents-play-the-long-game-point-2.svg)
*图示：机制化的难世界设计的概念示意*

- 怎么做：模拟器对26个客户群、群内每个客户独立建模，价格-质量决策遵循微观经济学的参与规则；包含正态、泊松、伯努利、对数正态等多种随机过程；环境随宏观周期、竞争对手提质、客户偏好漂移而非平稳演化；客户满意度、支付意愿等关键变量对Agent隐藏，只能通过数据库、社媒、谈判记录间接推断。
- 为什么 work：之前的Vending-Bench用LLM扮演供应商，Agent靠夸口承诺就能骗到资源，缺乏机制约束。CEO-Bench坚持用确定性公式+随机分布生成结果，使得Agent必须真正理解因果，而不是话术哄裁判。同时多群体相互影响、效果延迟，单点提升某个指标会被其他维度反噬，逼Agent考虑全局。
- 例子：Agent涨价后，某企业客户群体的口碑下降会通过reputation传导到相邻群体，几周后才在新客户获取速率上体现出来；同时一个R&D项目可能要数周才出成果且收益带噪，Agent必须从数据中反推这种延迟因果。

#### 技术点 3：可编程Python工具接口
Agent通过novamind\_api写脚本+SQL执行决策，而不是逐个调用工具

![可编程Python工具接口 理解图](assets/figures/tech-points/ceo-bench-can-agents-play-the-long-game-point-3.svg)
*图示：可编程Python工具接口的概念示意*

- 怎么做：动作空间通过Python包novamind-api暴露，Agent在终端里写脚本调用34个工具，参数支持细粒度结构化输入（如按(渠道,客户群)分配广告预算、按客户ID发放促销）。同时可对19张业务表执行SQL查询，把分析与执行串成自定义工作流。
- 为什么 work：把工具组合的自由度交给Agent写代码，比预定义固定流程更能暴露真实的规划与编排差异。强模型可以构造队列模拟、批量定价、按SQL结果定向营销，弱模型则只会重复单点工具调用，这种差异在传统'工具调用次数'类评测里看不到。
- 例子：GPT-5.5在第133天写脚本SQL拉取所有企业谈判历史，按customer group和plan聚合接受/拒绝价格分布，反推每个企业群的隐藏支付意愿；Claude Opus 4.8在第77天写代码模拟未来不同获客转化场景下的现金流，给出1/4/12/26周的预测矩阵后再决定策略。

#### 技术点 4：SOTA模型大面积翻车
10款主流模型只有Claude Opus 4.8和GPT-5.5突破百万起始资金，远低于规则基线之上的潜力上限

![SOTA模型大面积翻车 理解图](assets/figures/tech-points/ceo-bench-can-agents-play-the-long-game-point-4.svg)
*图示：SOTA模型大面积翻车的概念示意*

- 怎么做：每模型跑3次（seed=42）。最终现金：Claude Opus 4.8 27.8M、GPT-5.5 21.3M，规则基线15.76M，其余模型要么低于起始资金，要么破产；估算理论上限约2.2B。Grok 4.20、DeepSeek V4 Pro、Gemini 3 Flash、Claude Haiku 4.5全部三次跑全破产。
- 为什么 work：这说明短程Benchmark上看不出差距的模型，在长程经营里差异巨大；而且即使是最强模型，也远未触及理论上限，benchmark远未饱和。一个'什么都不学的固定规则'反而能稳定盈利1500万美元，揭示当前Agent在长程下经常做出比'啥都不变'还差的决策。
- 例子：Claude Opus 4.7选择保守的'harvest & die'策略，第77天起几乎冻结所有支出，最终勉强活到500天但只剩39万；而Claude Opus 4.8不断切换战略——先扩张S1/S2客户群，中期突然清零客户切到收割模式，靠一次正确的pivot做到2780万。

#### 技术点 5：四维能力归因分析
把'谁赢谁输'拆解到隐藏信息推断、未来预测、适应速度、规划深度四个量化指标

![四维能力归因分析 理解图](assets/figures/tech-points/ceo-bench-can-agents-play-the-long-game-point-5.svg)
*图示：四维能力归因分析的概念示意*

- 怎么做：作者用四个代理指标定量比较模型：(a)广告分配到最优渠道的比例（隐藏信息发现），(b)四周现金预测误差（前瞻），(c)首次竞争对手行动后memo中出现'competitor'词的延迟（适应），(d)memo中'if'出现频次（规划分支）。同时统计targeted dev spend占比衡量是否会用细粒度动作。
- 为什么 work：长程任务表现是综合产物，但作者用这些可观测代理指标说明'强模型强在哪'：Opus 4.8和GPT-5.5在四个维度上都明显领先，targeted dev占比近90%而其他模型只有10-43%。这给Agent研究提供了可分项归因的诊断思路，而不是只看最终分数。
- 例子：在5个广告渠道下，随机猜的最优渠道分配率是20%；Opus 4.8达到约43%，GPT-5.5约33%，其他模型大多低于20%——意味着多数Agent投广告基本接近瞎投，没有从历史数据中学到渠道-客户群匹配。

- **对 Agent 产品/系统的启发：**
  - 产品侧：做经营/运营/财务类Agent产品时，不能只测短程任务通过率，应构建多周以上的端到端模拟，关注现金流或KPI的长期轨迹，而非单步决策正确率；同时给Agent提供数据库+SQL+脚本化执行能力，比堆叠固定工具链更能发挥强模型潜力。
  - 系统侧：在Agent harness层，长程任务的上下文管理（周级clear+memory文件）比单纯加大context window更重要；动作接口设计应允许细粒度参数和代码组合，这样强模型才能写出cohort模拟、数据挖掘等高阶workflow。评测时建议拆分隐藏信息发现、未来预测、适应速度、规划分支四个子指标做归因。
  - 风险：弱模型在长程任务里会陷入'保守冻结'或'反复切换'两种失败模式，前者表面上不出错但毫无价值产出，后者破产；产品上线前应有破产/止损保护机制，并警惕LLM-as-judge的模拟器被Agent口头承诺欺骗，要尽量用机制化规则。

### 3. [必读 · 代码] Code-Augur: Agentic Vulnerability Detection via Specification Inference
- **arxiv 信息：** `2606.18619` · 作者：Zhengxiong Luo等 · 类目：cs.AI · 提交：2026-06 · [原文](https://arxiv.org/abs/2606.18619) · [PDF](https://arxiv.org/pdf/2606.18619.pdf)
- **为什么读：** 把Agent的隐含假设变成可证伪的源码断言，用fuzzer持续打脸
- **背景：** 当前LLM Agent已经能在Linux内核等大型项目里挖出潜伏多年的漏洞，但当Agent判定一段代码'安全'时，它基于的隐含假设既不可见也无人验证——可能漏掉关键corner case。已有方法（如Claude Code）只在Agent怀疑有bug时才动态验证，对于'判定安全'的部分完全不做检查，导致漏报无法被发现，审计结论也难以信任。
![Code-Augur: Agentic Vulnerability Detection via Specification Inference 论文主图](assets/figures/overview/code-augur-agentic-vulnerability-detection-via-specification-inference-hero.svg)
*图示：论文核心机制概念图*


**核心技术点**

#### 技术点 1：把假设写成源码断言
Agent判定安全时，把支撑该判断的不变量作为assert直接插入源代码

![把假设写成源码断言 理解图](assets/figures/tech-points/code-augur-agentic-vulnerability-detection-via-s-point-1.svg)
*图示：把假设写成源码断言的概念示意*

- 怎么做：Agent在Invariant Analysis阶段，对每个被判定为安全的位置，提取支撑这一判断的局部不变量ϕ（如 fmt.channels == cs.channels），并以原生断言形式（C/C++用sanitizer trap，Java用Jazzer通道等）写入源代码，重新构建为P'。每个断言带唯一ID，运行时违反可被结构化识别。
- 为什么 work：Agent的'我觉得这里安全'本来是一句模糊的自然语言推理，难以验证也难以复用。把它落到源码assert上，就变成了一个对所有路径都必须成立的硬性义务，而不是只对Agent走过的那条路径成立。同时不需要发明新的规约语言，复用项目原有概念，省事且不丢信息。
- 例子：在Little CMS例子里，Agent看到IsProperColorSpace有一条PT-ANY的wildcard分支后，仍认为CreateTransform安全。Code-Augur让Agent把支撑这个判断的不变量 fmt.channels == cs.channels 作为assert写进CreateTransform，这个断言变成所有进入该函数的路径都必须满足的契约。

#### 技术点 2：fuzzer去证伪Agent
用grey-box fuzzer专门攻击这些断言，比让Agent自己验证更不容易共享盲点

![fuzzer去证伪Agent 理解图](assets/figures/tech-points/code-augur-agentic-vulnerability-detection-via-s-point-2.svg)
*图示：fuzzer去证伪Agent的概念示意*

- 怎么做：Code-Augur把每个断言接入fuzzer的反馈通道：除了code coverage，还导出'是否到达断言、是否触发、距离触发多近'三类信号（C/C++用libFuzzer的extra-counter做对数距离分桶，JVM用Jazzer的exploreState/minimize）。fuzzer的目标是找一个输入i¬ϕ让ΠP(i¬ϕ)≠ΠP'(i¬ϕ)。
- 为什么 work：如果让Agent自己验证自己的假设，盲点会重复。用fuzzer做随机搜索可以补上语义直觉看不到的corner case。同时反过来，fuzzer不再盲目找崩溃，而是有了一个语义更浅、更容易触达的目标——破坏不变量，比直接撞到深处的内存错误容易得多。
- 例子：fuzzer不必直接构造一个能触发UnpackPixel越界读的复杂输入，只需找到一个让 fmt.channels != cs.channels 在CreateTransform就成立的profile（13 vs 1），断言立刻被打破，暴露出PT-ANY绕过路径。

#### 技术点 3：reason-falsify-refine闭环
fuzzer证伪后，要么是真bug，要么暴露Agent理解错误并迭代修正

![reason-falsify-refine闭环 理解图](assets/figures/tech-points/code-augur-agentic-vulnerability-detection-via-s-point-3.svg)
*图示：reason-falsify-refine闭环的概念示意*

- 怎么做：Violation Triage模块用LLM复现违反场景并检查程序状态，输出二选一：(a)这是真实安全缺陷，进入bug候选池；(b)断言写得过强，反馈fb给REFINEINVARIANT，让Agent修正不变量再下一轮fuzz。整个循环以审计预算耗尽为终止条件。
- 为什么 work：断言被打破不一定是bug，也可能只是Agent的世界模型太天真。把这两种情况分开处理，使Agent的程序理解 ˆΠP 持续向真实行为 ΠP 收敛，类似于一种'对抗自己的世界模型'的在线学习——不靠权重更新，而靠把推理产物变成可执行物再被反例修正。
- 例子：如果fuzzer找到一个让fmt.channels != cs.channels的输入但实际后续完全没触发任何越界，triage判定断言过强，Agent就会把不变量改成更精确的形式（例如限定到某些colormodel下才必须相等），下一轮再被fuzz。

#### 技术点 4：实测效果与新漏洞
在AIxCC和OSV基准上比SOTA Agent多挖34%–370%漏洞，并发现22个真实新漏洞

![实测效果与新漏洞 理解图](assets/figures/tech-points/code-augur-agentic-vulnerability-detection-via-s-point-4.svg)
*图示：实测效果与新漏洞的概念示意*

- 怎么做：在AIxCC基准（39个seeded漏洞）和OSV基准（24个真实CVE）上对比Claude Code和AIxCC冠军系统Atlantis，分别用Claude Sonnet 4.6和DeepSeek V4 Pro两个底座，每个挑战4小时、$100预算。Code-Augur比基线多发现34%–370%的bug，并在真实开源项目中找到22个未知漏洞，其中16个被开发者修复或确认。
- 为什么 work：证明这种'让Agent写下假设再被自动证伪'的范式不仅是理论上漂亮，在工业级项目（nginx、systemd、wireshark、apache-poi等）里实际多挖出来的漏洞确实更多。而且不依赖Claude Mythos这种专用模型，用通用的Sonnet/DeepSeek就能做到。

- **对 Agent 产品/系统的启发：**
  - 产品侧：代码审计/AI security类产品不应只输出bug列表，应同时沉淀'Agent当时认为成立的假设'作为可复用资产，跨版本、跨模块复检；当某次fix后断言再次被打破，可以直接定位到一类回归。
  - 系统侧：构建Agent系统时，可以把'Agent的判断 + 支撑该判断的可执行检查'作为基本输出单元，并配一个独立验证器（fuzzer/property test/形式化检查器）持续证伪，避免Agent自己验证自己的盲点共享问题。这是把'self-consistency'升级为'external falsification'。
  - 风险：断言可能写得过强或过弱：过强会引发大量benign violation浪费预算，过弱则形同虚设；refine循环的收敛性依赖triage判定的准确率，triage本身仍是LLM，仍有错判风险。

## 三、总结

- 今天415篇里，评测、安全、harness三条线都在向'更长、更真、更可验证'方向收敛。SafeClawBench揭示语义通过的样本里有84%在沙箱里仍然出事，CEO-Bench让多数SOTA模型在500天经营中破产或跑不赢固定规则，这两个反直觉结果共同说明现有Agent评测严重高估了真实能力。Code-Augur则给出一个具体可复制的范式——把Agent的隐式判断写成断言交给fuzzer证伪，可能是接下来Agent可信化的重要工具。
