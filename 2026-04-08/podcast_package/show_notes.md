# 04-08 AI Agent 速报：Harness（脚手架）是什么

- 日期：2026-04-08
- 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC

## 本期看点

- 今天初筛最明显的信号不是“再来一个更强 Agent”，而是评测基础设施在快速补课：agent_eval 虽然数量不占绝对多数，但高分论文密集落在 workspace、software environment、可控难度与长时程任务上，说明研究重点正从能力展示转向可复现、可比较、可上线前验证的系统测试。
- Agent 安全研究正在从传统 prompt 层防护外扩到整条决策链路：高分工作同时覆盖推理劫持、生产力场景安全、multi-agent fuzzing 以及机器人 action model 越狱，表明社区已把风险理解为“记忆-规划-工具-执行”联合暴露面，而不是单一输入输出问题。
- Computer-use 与 GUI/软件操作 Agent 进入“闭环执行”阶段：从把任意软件变成环境，到动作效果验证、意图条件化评估，研究判断已从“能不能给出动作”转向“动作是否真的生效、失败后能否恢复”，这意味着鲁棒执行正成为落地分水岭。
- 动作空间与技能组织开始结构化，反映 Agent 工程正从自由文本控制走向软件工程化：无论是 structured action spaces 还是 dependency-aware skill retrieval，都说明系统瓶颈越来越多出在动作抽象、技能编排与脚手架设计，而不只是底座模型本身。

## 重点论文

### 1. ClawsBench: Evaluating Capability and Safety of LLM Productivity Agents in Simulated Workspaces
- 方向：agent_eval
- 为什么值得关注：高相关Agent评测基准，覆盖生产力Agent能力与安全，在模拟工作空间中测真实任务，明显有助于后续系统设计与比较。
- arXiv：https://arxiv.org/abs/2604.05172
- PDF：https://arxiv.org/pdf/2604.05172.pdf

### 2. Don't Act Blindly: Robust GUI Automation via Action-Effect Verification and Self-Correction
- 方向：computer_use
- 为什么值得关注：直指GUI自动化Agent的鲁棒性，引入动作效果验证与自纠机制，极具落地性和产品相关性。
- arXiv：https://arxiv.org/abs/2604.05477
- PDF：https://arxiv.org/pdf/2604.05477.pdf

### 3. Stop Fixating on Prompts: Reasoning Hijacking and Constraint Tightening for Red-Teaming LLM Agents
- 方向：agent_safety
- 为什么值得关注：聚焦LLM Agent红队与推理劫持，切中Agent安全核心薄弱点，对防御与评测都很重要。
- arXiv：https://arxiv.org/abs/2604.05549
- PDF：https://arxiv.org/pdf/2604.05549.pdf
