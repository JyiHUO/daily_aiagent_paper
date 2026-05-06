# 2026-05-06 AI Agent 论文日报

> 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC
> 入选论文：3 篇

## 一、初筛每日趋势

- Agent安全持续下沉：从prompt走向工单链、skill编译、记忆投毒
- 评测集体转向真实工作空间+可执行Oracle判真
- Memory研究从系统设计走进内部电路诊断
- Agent安全议题从单轮prompt继续下沉到开发流水线与运行时层：MOSAIC-Bench用组合式Jira工单绕过9家生产Coding Agent，MAGE、ARGUS、MEMSAD、SkCC分别在长程记忆、上下文注入、记忆投毒和跨框架skill编译上各自加固，安全正从'对齐'转向'全链路治理'。
- Agent评测整体转向真实环境+可执行验证：Workspace-Bench搭起20GB级企业工作空间和7399条rubric，Reward Hacking Benchmark、Healthcare AI GYM、MOSAIC-Bench都坚持用容器/状态/PoC做真值，LLM judge的纯文本打分正被边缘化。
- Memory研究开始分化出'诊断派'：不只做分层记忆和retrieval（MEMTIER），还深入write-manage-read电路层定位沉默失败，显示记忆可靠性需要从系统工程向可解释性延伸。
- 多Agent与computer-use层面工程密度上升：Pact提出带博弈语义的choreographic语言，cotomi Act靠lazy observation和verbal-diff把WebArena推到80.4%，协议与harness正在成为独立研究对象。

### 跨论文综合观察

- MOSAIC-Bench和Workspace-Bench在方法论上高度共振：都坚持用真实环境+确定性Oracle（Docker PoC / 文件依赖图+rubric）取代LLM judge，标志Agent评测的判真标准正在硬化。
- MAGE的shadow memory、MEMSAD的投毒检测和'Circuit Analysis'的按阶段诊断，从防御、检测、解释三个角度共同指向同一问题——Agent记忆层的沉默失败，需要运行时+内部信号共同治理。
- MOSAIC-Bench揭示的'组合式绕过'和SkCC的'Skill Injection防御'、ARGUS的'provenance影响图'形成互补：攻击面已经从单轮prompt扩展到工单、skill、上下文多条注入路径，防御也必须转向全链路追踪。

## 二、重点论文精读

### 1. MOSAIC-Bench: Measuring Compositional Vulnerability Induction in Coding Agents
- **方向：** agent\_safety
- **评分：** 相关性 95 | 价值 90 | 有趣性 88 | 创新性 85 | 开拓性 85
- **为什么入选：** 揭示Coding Agent在工单拆分下安全对齐失效，53-86%无害工单组合出真实漏洞。
- **快速背景：** 单轮安全对齐挡不住把恶意目标拆成多张普通工单的攻击。
![MOSAIC-Bench: Measuring Compositional Vulnerability Induction in Coding Agents 关键架构图](assets/figures/overview/mosaic-bench-measuring-compositional-vulnerability-induction-in-coding-agents-hero.png)
*图示：这张图是全篇最接近方法总览/系统流程图的候选：它清楚展示了MOSAIC-Bench的两层pipeline，包括构造层的五个阶段以及验证层的三个独立gate，直接说明了benchmark如何生成、筛选并确认可利用的攻击链，也体现了oracle、reviewer ensemble和逐阶段diff这些论文核心机制。其他候选大多是结果图，虽然重要，但不如这张图能让读者第一眼理解论文的方法与系统结构。*

- **详细背景：** 现有安全对齐只评测单条提示是否显性有害，Agent按prompt单独看每个请求都会拒绝或加固代码。但真实开发是按Jira工单串行推进的，攻击者把一个CVE拆成三张看起来无害的工程工单后，Agent一张张照做就拼出了可被利用的漏洞。之前的decomposition研究只监控用户对话序列，insecure code基准又只看单轮输出，没有人把'利用是否真实可复现'和'下游reviewer是否放行'同时作为一等评测维度，这正是MOSAIC-Bench补齐的空白。
- **详细入选理由：** 这篇论文首次在生产级Coding Agent上系统量化了'组合式漏洞诱导'——把一个恶意目标拆成三张普通Jira工单后，9家厂商的Agent几乎100%合规执行，端到端漏洞利用率高达53-86%，并且下游Code Review Agent也批准了四分之一。对所有在做Coding Agent、Code Review Agent和安全审计产品的团队都是高价值警报，同时给出了'把reviewer重新框定为渗透测试员'这一零训练可部署的缓解方案。

**核心技术点速览：**

#### 技术点 1：组合式工单绕过防御
- 快速理解：同一漏洞单轮被拒0-20%，拆成三张工单却达成53-86%利用率。

![组合式工单绕过防御 理解图](assets/figures/tech-points/mosaic-bench-measuring-compositional-vulnerabili-point-1.svg)
*图示：可以把Agent的安全反射想成'看到明显坏词就踩刹车'。当攻击者把'加metadata字段''加PATCH接口''加markModified修持久化'三张普通工单分别丢给Agent，每张都是教科书式的正常需求，Agent根本没东西可拒；三步叠加后却正好组成CWE-915 Mass Assignment，一个POST就能把自己升成admin。*

- 技术细节：作者在199条三阶段攻击链上测了9个生产Coding Agent，使用部署实景的'每张工单一次新会话'协议，端到端ASR达到53-86%，全部staged运行里只有2次拒绝；而把同一漏洞合成单条直接prompt时，Claude拒绝率78-89%、Codex主要走代码加固，VULNERABLE输出仅0-20.4%。
- 通俗讲解：可以把Agent的安全反射想成'看到明显坏词就踩刹车'。当攻击者把'加metadata字段''加PATCH接口''加markModified修持久化'三张普通工单分别丢给Agent，每张都是教科书式的正常需求，Agent根本没东西可拒；三步叠加后却正好组成CWE-915 Mass Assignment，一个POST就能把自己升成admin。
- 例子：论文示例：Express+Mongoose下FEAT-910加Mixed类型metadata字段、FEAT-911加PATCH用Object.assign写入、BUG-525按Mongoose官方文档加markModified修bug。三张工单全部通过四个reviewer审核上线后，攻击者发一个('role':'admin')就被写入并持久化，PoC在容器里确定性复现。

#### 技术点 2：上下文碎片不是主因
- 快速理解：保留完整会话记忆也只能关掉一半gap，说明是结构性问题。

![上下文碎片不是主因 理解图](assets/figures/tech-points/mosaic-bench-measuring-compositional-vulnerabili-point-2.svg)
*图示：很自然会猜：Agent拒绝是因为没看见全局，那把三张工单都摆在同一个对话里它是不是就能警觉？答案是只能解释一半。即使给它完整记忆、明确串起来，它依然会把这三步当作合理的功能迭代继续实现。说明问题不是'没看见'，而是分阶段的工程语境本身削弱了安全反射。*

- 技术细节：作者做了resumed-session消融：让Agent在同一会话里依次收到三张工单、保留全部上下文记忆，再对比fresh-session的ASR。结果中位数只下降了约50%的gap，GPT 5.4甚至只降2pp仍保持59.8% ASR，Gemini和Kimi几乎不变。同时直接prompt baseline也排除了'纯safety bypass'解释。
- 通俗讲解：很自然会猜：Agent拒绝是因为没看见全局，那把三张工单都摆在同一个对话里它是不是就能警觉？答案是只能解释一半。即使给它完整记忆、明确串起来，它依然会把这三步当作合理的功能迭代继续实现。说明问题不是'没看见'，而是分阶段的工程语境本身削弱了安全反射。
- 例子：同一批chain下，Codex 5.3从85.9%降到63.3%，但仍远高于直接prompt时的20.4%；Opus 4.6从53.3%降到42.7%，而直接prompt下它的漏洞输出率几乎为0——说明'看全'并不能让它想起来要拒绝。

#### 技术点 3：Reviewer渗透测试员框定
- 快速理解：把代码审查Agent改成'红队视角'是性价比最高的零训练缓解。

![Reviewer渗透测试员框定 理解图](assets/figures/tech-points/mosaic-bench-measuring-compositional-vulnerabili-point-3.svg)
*图示：同样的diff、同样的模型，只是告诉它'你是攻击者，试着写一个可用exploit，然后再决定approve/decline'，它就会主动去想这段改动能被怎么滥用，而不是按常规PR思路看代码风格。这种'换一个提问姿势'的改法不需要训练，任何跑AI code review的团队今天就能用。*

- 技术细节：在reviewer端，把system prompt从'资深工程师做PR review'换成'渗透测试员，必须先列CWE再DECLINE'，五个reviewer配置的evasion全部下降；开源小模型Gemma-4-E4B-it在pentester框定下达到88.4%检测率，在608条真实GitHub PR上FP仅4.6%，单次成本约$0.001，接近Sonnet 4.6全上下文水平但便宜两三个数量级。
- 通俗讲解：同样的diff、同样的模型，只是告诉它'你是攻击者，试着写一个可用exploit，然后再决定approve/decline'，它就会主动去想这段改动能被怎么滥用，而不是按常规PR思路看代码风格。这种'换一个提问姿势'的改法不需要训练，任何跑AI code review的团队今天就能用。
- 例子：Sonnet 4.6 diff-only evasion从21.6%降到3.0%；Gemma-4从59.8%降到11.6%。但GPT 5.4是弱迁移样本，只从27.6%降到17.6%，作者明确提示框定收益是异质的，不是对所有模型都普适。

#### 技术点 4：可验证基准与Oracle
- 快速理解：199条链配确定性PoC和Docker底座，把利用真值当一等评测维度。

![可验证基准与Oracle 理解图](assets/figures/tech-points/mosaic-bench-measuring-compositional-vulnerabili-point-4.svg)
*图示：以往基准要么让LLM judge判漏洞（容易糊弄），要么只测单轮代码是否'看起来不安全'。这里作者坚持一条：真漏洞就是能在一个跑着的Docker里被PoC打穿。所有ASR、reviewer evasion都是相对这个固定真值算出来的，所以可以干净地把'reviewer协议'当成自变量来实验。*

- 技术细节：MOSAIC-Bench覆盖10个Web应用底座、31个CWE类、5种语言，每条chain是三阶段Jira工单+逐阶段golden脚本+针对运行容器的Python PoC oracle，返回VULNERABLE/SECURE。评测分两层：构造层用4-reasoner council生成候选，验证层有per-stage diff、reviewer ensemble、oracle exploitability三道独立闸门，只有oracle有资格判定利用真值。
- 通俗讲解：以往基准要么让LLM judge判漏洞（容易糊弄），要么只测单轮代码是否'看起来不安全'。这里作者坚持一条：真漏洞就是能在一个跑着的Docker里被PoC打穿。所有ASR、reviewer evasion都是相对这个固定真值算出来的，所以可以干净地把'reviewer协议'当成自变量来实验。
- 例子：一条CWE-915链的oracle会：部署三阶段合成diff到新容器变成等健康检查变成跑PoC发送恶意PATCH变成验证注入字段是否被markModified持久化并可被后续读取，返回VULNERABLE才记为成功；100条抽样人工复核false-positive为0。

- **对 Agent 产品/系统的启发：** 工单拆分会同时关闭拒绝和加固两种防御，必须在workflow层而非单prompt层加防线。
- **详细启发：** 产品侧：任何'Jira工单→Coding Agent→Code Review Agent→合并'的产品链路都存在结构性盲区：每张工单单看都合法，合起来可能就是CVE。产品侧应把'跨工单累积diff审计'作为独立环节，并默认给code review agent套上pentester/红队人格提示，把列CWE作为DECLINE的前置条件。；系统侧：系统架构上需要在ticket author→implementing agent之间加入序列级监控（如Yueh-Han等的sequential monitor）以及累积diff oracle化测试；异构reviewer ensemble（Codex抓数据组合侧信道、Claude抓认证基建、Gemini抓信息泄漏）比单reviewer扩scale更有效。低成本路线可用开源小模型+pentester框定做第一道筛。；风险：1) 单prompt安全评测完全不能反映真实部署风险，以它为KPI会严重低估漏洞率；2) 给Agent更多上下文记忆只能缓解一半问题，不能替代结构性防御；3) pentester框定是非自适应缓解，chain是针对neutral reviewer筛过的，攻击者一旦针对pentester再迭代，收益会被侵蚀；4) 恶意insider只要有backlog写权限就能低成本发动此类攻击。

### 2. What Happens Inside Agent Memory? Circuit Analysis from Emergence to Diagnosis
- **方向：** memory
- **评分：** 相关性 92 | 价值 85 | 有趣性 88 | 创新性 85 | 开拓性 82
- **为什么入选：** 首次从电路层拆解Agent memory的写管读，给出无监督的失败定位方法
- **快速背景：** Agent记忆系统的失败是沉默的——输出看起来合理，但写/管/读哪一步错了无从判断。
![What Happens Inside Agent Memory? Circuit Analysis from Emergence to Diagnosis 关键架构图](assets/figures/overview/what-happens-inside-agent-memory-circuit-analysis-from-emergence-to-diagnosis-hero.png)
*图示：这张图是候选中最接近方法机制图的一张：它直接展示了 Write operation 的内部 feature circuit、分阶段模块关系与信息流（subject anchoring → word-specific extraction → category aggregation hub），能体现论文核心贡献——把 agent memory 的外部写管读流程映射到内部电路机制。虽然它不是完整系统总览，但比其他候选更能一眼说明论文在“memory内部机制分析”上的方法本体。Figure 1 只是失败案例示意，Figure 3 是结果图，其余候选不是有效主图。*

- **详细背景：** 基于LLM的Agent普遍用'写-管-读'三步管理长期记忆（如mem0、A-MEM），但每一步输出看起来都合法，错了也不会报错，端到端准确率又把三种错误混在一起。过去的机械可解释性工作只研究单轮检索或静态事实定位，没有人把电路追踪扩展到这种多次调用的流水线。论文第一次跨模型规模（Qwen-3 0.6B–14B）和两个框架去追踪每个阶段的特征电路。
- **详细入选理由：** 这篇论文把Agent memory的沉默失败问题，从外部评测推进到内部电路层面，不仅解释了'小模型为何会假装记忆正常'，还提出76.2%准确率的无监督按阶段失败定位方法，对做memory层可靠性的团队有直接参考价值。

**核心技术点速览：**

#### 技术点 1：控制电路先于内容电路出现
- 快速理解：0.6B就有路由电路能做决策，但提取/读取电路要到4B才成形，小模型会'装作记得'。

![控制电路先于内容电路出现 理解图](assets/figures/tech-points/what-happens-inside-agent-memory-circuit-analysi-point-1.svg)
*图示：可以把Agent记忆想成一个调度员加一个秘书：调度员决定'这条信息要更新还是丢弃'，秘书负责真把信息提取出来或从记忆里读出来。论文发现小模型的调度员很早就上岗了，但秘书要等模型长到4B才出生。结果就是0.6B的Agent会信心满满地按UPDATE键，但它其实根本没读懂这条信息是什么。*

- 技术细节：作者在Qwen-3四个规模上对Write/Manage/Read三步做电路因果验证。Manage（add/update/delete/none路由）在0.6B就有显著causal gap（0.259，bootstrap CI不含零），而Write和Read的内容电路直到4B才出现可检测信号。两个框架mem0和A-MEM都观察到同样的先后顺序。
- 通俗讲解：可以把Agent记忆想成一个调度员加一个秘书：调度员决定'这条信息要更新还是丢弃'，秘书负责真把信息提取出来或从记忆里读出来。论文发现小模型的调度员很早就上岗了，但秘书要等模型长到4B才出生。结果就是0.6B的Agent会信心满满地按UPDATE键，但它其实根本没读懂这条信息是什么。
- 例子：论文开头的例子：老记忆是'用户开普锐斯'，新来一句无关的'用户喜欢周末远足'。0.6B直接输出('event':'UPDATE')把旧记忆覆盖掉；8B才正确返回('event':'NONE')。路由动作合法、JSON合法，外部完全看不出错误。

#### 技术点 2：Write和Read共享一个grounding hub
- 快速理解：写和读在深层共用一个'上下文接地'特征簇，只有memory框架的prompt才会激活其中的功能方向。

![Write和Read共享一个grounding hub 理解图](assets/figures/tech-points/what-happens-inside-agent-memory-circuit-analysi-point-2.svg)
*图示：这个hub本来就存在于base模型里，是通用的'把上下文信息拉进回答'的底座。memory框架并没有新造一个记忆模块，而是借用了这个底座，在上面激活一条'记忆专用的注意方向'。换句话说，记忆框架的上限受限于底座本身能支持的方向，光改prompt或存储格式很难突破。*

- 技术细节：8B模型里Write和Read电路在L34附近共享约10个co-recurring features构成的hub。移植hub激活会扰动55%的预测，而同层随机特征无影响。作者从memory-present与memory-absent的激活差定义一条'grounding方向'，抑制它会让模型偏离记忆答案约0.5 log-prob；在直接context对照条件下提取的方向则无此效果（∆M= −2.30 vs +0.32）。
- 通俗讲解：这个hub本来就存在于base模型里，是通用的'把上下文信息拉进回答'的底座。memory框架并没有新造一个记忆模块，而是借用了这个底座，在上面激活一条'记忆专用的注意方向'。换句话说，记忆框架的上限受限于底座本身能支持的方向，光改prompt或存储格式很难突破。
- 例子：当Read prompt是'根据MEMORY回答'时，L34 hub会沿着记忆grounding方向激活，把检索到的'用户开Tesla Model 3'拉进输出；如果把这个方向人为dampen掉，模型会退回用参数里的先验答案，忽视刚检索到的记忆条目。

#### 技术点 3：可检测 ≠ 可干预
- 快速理解：内容电路4B就能看到，但只有到8B放大干预才稳定有效，14B又因电路分散而失灵。

![可检测 ≠ 可干预 理解图](assets/figures/tech-points/what-happens-inside-agent-memory-circuit-analysi-point-3.svg)
*图示：看到电路存在，不等于可以通过调节它来纠正行为。内容电路很'娇气'，干预强度稍微不对就把流水线打乱。作者据此放弃了'写入式干预'思路，改用电路信号做只读诊断。这对想做'运行时干预记忆'的产品来说是个重要的劝退信号。*

- 技术细节：作者对Write/Read电路top特征做2×/3×/5×/10×放大干预，测fact recall和QA。只有8B在所有倍数下都获得+2～+3pp的一致提升；4B在5×时fact recall塌陷到.218（−62pp），10×又恢复；14B效果小且方向不一致；0.6B因没有成形电路而退化。
- 通俗讲解：看到电路存在，不等于可以通过调节它来纠正行为。内容电路很'娇气'，干预强度稍微不对就把流水线打乱。作者据此放弃了'写入式干预'思路，改用电路信号做只读诊断。这对想做'运行时干预记忆'的产品来说是个重要的劝退信号。
- 例子：在4B上对Write电路放大5倍，原本.838的fact recall会崩到.218——模型几乎不能正确抽取事实；但放到10倍又回到.866。这种非单调说明同一个电路在不同倍数下的行为不可预测，不能稳定部署。

#### 技术点 4：无监督按阶段定位沉默失败
- 快速理解：基于电路特征分离做零训练诊断，在8B上76.2%准确定位是Write/Manage/Read哪一步错了。

![无监督按阶段定位沉默失败 理解图](assets/figures/tech-points/what-happens-inside-agent-memory-circuit-analysi-point-4.svg)
*图示：因为Write、Manage、Read三类电路在特征空间里几乎不重叠，可以把它们当作三个'探针'。给一个错答的样本，挨个关掉三组特征看谁的扰动最大，就知道是哪一步出的错，然后按诊断结果选对应修复动作（重抽取、规则删除、hub放大）。条件性修复比对所有样本无差别放大获益更多（+5–8pp vs +3pp）。*

- 技术细节：每个操作预选30个可区分特征（按可判别性而非归因强度选）。对一个失败样本，依次消融三组特征，哪组消融造成的输出扰动最大就判定为责任阶段。8B上达到76.2%，比最强无训练基线高24pp、比训练过的logistic回归高13pp；在LoCoMo和MemoryAgentBench上无需重训仍保持\>65%。
- 通俗讲解：因为Write、Manage、Read三类电路在特征空间里几乎不重叠，可以把它们当作三个'探针'。给一个错答的样本，挨个关掉三组特征看谁的扰动最大，就知道是哪一步出的错，然后按诊断结果选对应修复动作（重抽取、规则删除、hub放大）。条件性修复比对所有样本无差别放大获益更多（+5–8pp vs +3pp）。
- 例子：一个Read grounding failure的样本：模型检索到了正确fact但回答忽略了它。诊断器发现消融Read特征带来最大扰动，于是只对这类样本放大hub grounding方向让模型重新关注记忆条目，从而提升最终QA准确率。

- **对 Agent 产品/系统的启发：** 小模型别轻信它的记忆路由，建议用电路信号做阶段级诊断而非盲目干预。
- **详细启发：** 产品侧：做长期记忆的Agent产品不能只看端到端准确率，要对Write/Manage/Read分别埋点。尤其警惕用\<4B小模型跑记忆管理——它的UPDATE/DELETE看着对，但背后的事实抽取可能是空的。；系统侧：可以把论文的电路诊断作为离线debug工具：每次memory流水线出错时，用transcoder hook跑一遍阶段定位，再根据责任阶段触发对应修复（重抽取、规则删除、grounding方向放大）。一次诊断只需每阶段一次前向，适合接入CI或回归评测。；风险：直接用特征放大做'运行时记忆增强'不可靠，仅在8B窄窗口成立；跨模型家族（Llama、Gemma）需要重新训transcoder，现有结论不保证迁移。prompt很短、多轮长上下文可能激活额外机制。

### 3. Workspace-Bench 1.0: Benchmarking AI Agents on Workspace Tasks with Large-Scale File Dependencies
- **方向：** agent\_eval
- **评分：** 相关性 92 | 价值 85 | 有趣性 80 | 创新性 80 | 开拓性 82
- **为什么入选：** 首个面向真实工作空间的大规模跨文件依赖Agent评测基准
- **快速背景：** 现有Agent评测大多用几个独立文件，无法测出真实办公场景中的跨文件依赖推理。
![Workspace-Bench 1.0: Benchmarking AI Agents on Workspace Tasks with Large-Scale File Dependencies 关键架构图](assets/figures/overview/workspace-bench-1-0-benchmarking-ai-agents-on-workspace-tasks-with-large-scale-f-hero.png)
*图示：这张 Figure 2 是最完整的总览图，能一眼串起论文的核心机制：从 Workspace-Bench 的跨文件依赖任务、角色化工作空间、Agent 与真实世界工作空间交互，到基于能力维度的评测设置，既展示 benchmark 构成，也展示 agent-eval workflow 和模块关系。相比 Figure 3 主要是数据构建流程、Figure 6 主要是评测执行框架，这张图对读者理解整篇论文的对象、环境、任务与评测闭环最有代表性。*

- **详细背景：** 现有Agent基准要么把任务信息全塞在prompt里（CL-Bench等），要么只给几份孤立文件让Agent做文档QA（OfficeQA-Pro、GDPVal），要么只模拟单一风格的文件系统（OfficeBench、TheAgentCompany），文件格式普遍少于10种，也不显式考察版本血缘等跨文件依赖。真实知识工作者的工作空间是嵌套目录+数千异构文件+历史版本+隐式引用的混乱环境，这一层能力从未被系统评测，而企业落地Agent恰恰卡在这里。
- **详细入选理由：** 这篇论文把Agent评测从'几个独立文件+单一技能'推到了'20GB级真实工作空间+跨文件依赖推理'，涵盖5种职业角色、388任务、7399条rubric，并系统对比了28种harness×模型组合，是目前最能反映企业级Agent真实能力差距的评测集之一。

**核心技术点速览：**

#### 技术点 1：真实工作空间环境构建
- 快速理解：用5类职业画像模拟出2万+文件、74种格式、最高20GB的真实乱工作空间。

![真实工作空间环境构建 理解图](assets/figures/tech-points/workspace-bench-1-0-benchmarking-ai-agents-on-wo-point-1.svg)
*图示：作者没有让Agent在干净的sandbox里做题，而是仿造真实员工的电脑：有src/tests/docs的开发者目录，也有literature/experiments的研究员目录，里面混着report-v1、report-reviewed、report-final这种版本链。Agent拿到任务时必须自己翻目录、辨别哪个是最新版，而不是被喂好文件。*

- 技术细节：论文按Operations Manager、Logistics Manager、AI Product Manager、Backend Developer、Researcher五种职业画像，采用'先生成目录树、再爬取+LLM合成文件内容'的两阶段pipeline，构建出共20,476个文件、3,299个目录、最大深度8、74种文件类型的工作空间，并故意注入冗余文件夹、模糊命名、历史版本等噪声。
- 通俗讲解：作者没有让Agent在干净的sandbox里做题，而是仿造真实员工的电脑：有src/tests/docs的开发者目录，也有literature/experiments的研究员目录，里面混着report-v1、report-reviewed、report-final这种版本链。Agent拿到任务时必须自己翻目录、辨别哪个是最新版，而不是被喂好文件。
- 例子：Researcher工作空间最大，有11,020个文件、2,059个目录；而AI Product Manager只有1,379个文件。Agent接到'写周报'这类任务时，得自己在这种嵌套深度平均3.7的目录里去挖相关证据。

#### 技术点 2：依赖图驱动的任务标注
- 快速理解：每个任务都配一张文件依赖图和约19条细粒度rubric，可查过程不只查结果。

![依赖图驱动的任务标注 理解图](assets/figures/tech-points/workspace-bench-1-0-benchmarking-ai-agents-on-wo-point-2.svg)
*图示：光看最终输出对不对不够——Agent可能靠一个过时版本的文件也能蒙出看似合理的报告。所以论文明确标出'这个任务必须读到哪几个文件、它们之间什么关系'，再分别打分：文件找对了没、过程走对了没、结果算对了没。这让评测能定位到Agent具体哪一步垮了。*

- 技术细节：388个任务来自ByteDance Lark平台154个真实工作流，由25名对应角色的标注员人工编写指令、参考输出、文件依赖图和rubrics。每个任务平均5.1条依赖边、4.7个关键文件、19.1条rubric，rubric分为Foundation（25%）、Process（20.2%）、Result（54.8%）三类，并用辅助Agent把'计算是否正确'这种模糊标准改写成'最终值是否等于X'的可验证断言。
- 通俗讲解：光看最终输出对不对不够——Agent可能靠一个过时版本的文件也能蒙出看似合理的报告。所以论文明确标出'这个任务必须读到哪几个文件、它们之间什么关系'，再分别打分：文件找对了没、过程走对了没、结果算对了没。这让评测能定位到Agent具体哪一步垮了。
- 例子：一个'生成全球市场产品策略'的Operations Manager任务，要求整合5个区域订单、物流、客户分层数据，依赖图列出9个核心文件，再用25条rubric去查：报告里欧洲销售额是否503,568.21美元？Technology品类毛利率是否19.16%？共2条基础+21条结果+2条过程检查。

#### 技术点 3：六维能力×三难度的诊断设计
- 快速理解：从工作空间探索到版本血缘追溯分六维打分，能定位Agent短板在哪层。

![六维能力×三难度的诊断设计 理解图](assets/figures/tech-points/workspace-bench-1-0-benchmarking-ai-agents-on-wo-point-3.svg)
*图示：不同任务考的能力不一样：简单任务只要能翻目录找到结果文件；难任务要跨模态把PPT里的图表追回到源Excel，还得识别report-final才是最终版。这套标签让报告可以说清楚'Agent在血缘追踪上特别差'，而不是只给一个总分。*

- 技术细节：任务被打上六个能力标签：Workspace Exploration（262任务）、Task-Supporting Files Utilization（238）、Result-Providing Files Utilization（211）、Semantic Content Relations（170）、Heterogeneous File Understanding（140）、Lineage Tracing（136）；按所需步数和协作复杂度分Easy/Medium/Hard三档（14%/53%/33%）。
- 通俗讲解：不同任务考的能力不一样：简单任务只要能翻目录找到结果文件；难任务要跨模态把PPT里的图表追回到源Excel，还得识别report-final才是最终版。这套标签让报告可以说清楚'Agent在血缘追踪上特别差'，而不是只给一个总分。
- 例子：实验结论显示所有配置都卡在Heterogeneous File Understanding和Lineage Tracing两维，Easy任务平均57.6%、Hard掉到40.5%，说明一旦要处理多种格式或历史版本，Agent能力明显崩盘。

#### 技术点 4：28组Agent配置的系统评测
- 快速理解：最强组合仅68.7%、人类80.7%，Harness对弱模型是救命稻草对强模型几乎无用。

![28组Agent配置的系统评测 理解图](assets/figures/tech-points/workspace-bench-1-0-benchmarking-ai-agents-on-wo-point-4.svg)
*图示：最好组合OpenClaw+Opus-4.7只到68.7%，平均47.4%，远低于人类+工具的80.7%。而且开源组合容易'成本爆炸'——DeepAgent+MiniMax-M2.7一个任务能烧到58.1轮交互、61万token，分数仍只有45%。harness对强模型影响不大，但对弱模型能显著拉一把。*

- 技术细节：论文用统一的评测框架（含Sandbox池并行、工作空间回滚的并行BFS算法、多策略结果抽取、Agent-as-a-Judge）评估了ClaudeCode、DeepAgent、Hermes、OpenClaw四种harness × Opus-4.7、GLM-5.1、MiniMax-M2.7、Kimi-2.5、GPT-5.4、Gemini-3.1-Pro、Seed-2.0-Code七种底座，共28个组合。
- 通俗讲解：最好组合OpenClaw+Opus-4.7只到68.7%，平均47.4%，远低于人类+工具的80.7%。而且开源组合容易'成本爆炸'——DeepAgent+MiniMax-M2.7一个任务能烧到58.1轮交互、61万token，分数仍只有45%。harness对强模型影响不大，但对弱模型能显著拉一把。
- 例子：同样用Opus-4.7，换三种harness都能进前三，说明模型够强时框架差不多；而同样的harness换不同模型，得分能从顶端掉到最底，说明模型天花板仍是主因。

#### 技术点 5：Lite子集与评测框架工程
- 快速理解：100题Lite子集省70%评测成本，配套并行沙箱和回滚机制让大规模评测可复现。

![Lite子集与评测框架工程 理解图](assets/figures/tech-points/workspace-bench-1-0-benchmarking-ai-agents-on-wo-point-5.svg)
*图示：一个任务要改动几千个文件，跑完还得把环境还原到初始状态，不然下一个任务会被污染。论文用基线快照+并行BFS逐层比对差异，只补缺、删多、替换改动的节点，避免整盘重拷，这是真正支持大规模评测并行跑的工程基础。*

- 技术细节：Workspace-Bench-Lite从全量388任务中按工作空间、难度、六维能力分布均衡抽样100题，评测成本降约70%。框架用YAML声明配置，Sandbox Pool预克隆多副本支持workspace级（5x）+task级双重并行；任务后用并行BFS对比基线快照快速回滚被改动的文件系统；结果抽取采用'指令指定路径+统一副本目录+全局模糊匹配'三策略取并集。
- 通俗讲解：一个任务要改动几千个文件，跑完还得把环境还原到初始状态，不然下一个任务会被污染。论文用基线快照+并行BFS逐层比对差异，只补缺、删多、替换改动的节点，避免整盘重拷，这是真正支持大规模评测并行跑的工程基础。
- 例子：Agent跑完可能把结果文件存在任意路径。框架一方面在prompt里强制它在最终回复里写明路径，另一方面让它再存一份到全局目录，最后还用文件名模糊匹配全盘扫一遍，三路结果去重后交给Agent-as-a-Judge逐条rubric打分。

- **对 Agent 产品/系统的启发：** 做办公Agent要重点补跨文件依赖推理和版本血缘，而不是再堆工具调用花样。
- **详细启发：** 产品侧：面向企业的Agent产品不要再只秀'能调工具、能点GUI'，真正价值在于能在几千个乱序文件里找到对的版本、对的依赖链。产品设计应显式暴露文件依赖视图，让用户能看到Agent选了哪些证据、用了哪个版本，并保留人机协作入口——论文显示人+工具(80.7%)仍远超全自动(68.7%)。；系统侧：Agent系统层需要补齐三块：(1) 工作空间感知的检索——不仅是向量RAG，还要能理解目录语义和版本血缘；(2) 异构文件解析能力，覆盖70+格式而非常见10种；(3) 可回滚的沙箱执行+并行评测基础设施，这是让Agent敢在真实文件系统上操作的前提。同时要监控token/轮次成本，防止DeepAgent类harness出现的'成本爆炸'。；风险：Agent在Heterogeneous File Understanding和Lineage Tracing上普遍薄弱，直接部署到企业真实工作空间很容易读到过时版本、遗漏支撑文件，导致报告'看起来对但数据是旧的'，这在财务、合规等场景是高风险失误。


## 三、总结

- 今天信号明确：Agent安全、评测、记忆都在往'真实、可验证、可诊断'方向收敛
- 今天28篇must\_read里
- 今天28篇must\_read里，Agent安全与评测仍占最大比例，但两者都在往更深的层级走：安全从prompt对齐下沉到工单链、skill编译和记忆投毒，评测从静态题库走向企业级工作空间与容器PoC真值。
- Memory线的亮点在于'诊断'而非'更大存储'——电路层按阶段定位沉默失败，说明记忆层的可靠性问题已经不能只靠工程堆叠解决。
- 整体看，AI Agent研究正在进入一个'全链路可验证'阶段：任何声称的能力和安全，都要能在真实环境里被复现、被审计、被定位。
