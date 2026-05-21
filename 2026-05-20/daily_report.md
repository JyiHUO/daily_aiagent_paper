# 2026-05-20 AI Agent 论文日报

> 分类：cs.AI + cs.CL + cs.LG + cs.MA + cs.RO + cs.SE + cs.HC
> 入选论文：3 篇

## 一、初筛每日趋势

- 今天初筛后最集中的主题是 agent\_eval、tool\_use、…
- 高优先级论文里，EngiAI: A Multi-Agent Fra…
- 从创新性和研究开拓性看，EngiAI: A Multi-Agent…
- 今天初筛后最集中的主题是 agent\_eval、tool\_use、planning\_reasoning，说明研究关注点继续从单轮回答能力转向更完整的执行链。
- 高优先级论文里，EngiAI: A Multi-Agent Framework and Benchmark Suite for LLM-Driven Engineering Design 等工作都在强调 Agent 的系统边界，而不是只卷更大的底座模型。
- 从创新性和研究开拓性看，EngiAI: A Multi-Agent Framework and Benchmark Suite for LLM-Driven Engineering Design、RoboJailBench: Benchmarking Adversarial Attacks and Defenses in Embodied Robotic Agents 代表了今天最值得后续继续追踪的切口。

## 二、重点论文精读

### 1. EngiAI: A Multi-Agent Framework and Benchmark Suite for LLM-Driven Engineering Design
- **方向：** tool\_use
- **评分：** 相关性 100 | 价值 100 | 有趣性 100 | 创新性 100 | 开拓性 100
- **为什么入选：** 和 Agent 核心能力或系统设计直接相关，值得优先读。
- **快速背景：** EngiAI: A Multi-Agent Framework and Ben…
![EngiAI: A Multi-Agent Framework and Benchmark Suite for LLM-Driven Engineering Design 论文机制总览图](assets/figures/overview/engiai-a-multi-agent-framework-and-benchmark-suite-for-llm-driven-engineering-de-hero.svg)
*图示：候选主图不可靠，已回退为论文核心机制总览 SVG。*

- **当前状态：** llm_failed（LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}）
- **核心技术点：** 本次精读未成功，暂不展示结构化核心点，避免误导。

### 2. RoboJailBench: Benchmarking Adversarial Attacks and Defenses in Embodied Robotic Agents
- **方向：** embodied\_agent
- **评分：** 相关性 100 | 价值 100 | 有趣性 100 | 创新性 100 | 开拓性 100
- **为什么入选：** 和 Agent 核心能力或系统设计直接相关，值得优先读。
- **快速背景：** RoboJailBench: Benchmarking Adversarial…
![RoboJailBench: Benchmarking Adversarial Attacks and Defenses in Embodied Robotic Agents 论文机制总览图](assets/figures/overview/robojailbench-benchmarking-adversarial-attacks-and-defenses-in-embodied-robotic--hero.svg)
*图示：候选主图不可靠，已回退为论文核心机制总览 SVG。*

- **当前状态：** llm_failed（LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}）
- **核心技术点：** 本次精读未成功，暂不展示结构化核心点，避免误导。

### 3. STAR-PólyaMath: Multi-Agent Reasoning under Persistent Meta-Strategic Supervision
- **方向：** planning\_reasoning
- **评分：** 相关性 100 | 价值 90 | 有趣性 88 | 创新性 94 | 开拓性 89
- **为什么入选：** 和 Agent 核心能力或系统设计直接相关，值得优先读。
- **快速背景：** STAR-PólyaMath: Multi-Agent Reasoning u…
![STAR-PólyaMath: Multi-Agent Reasoning under Persistent Meta-Strategic Supervision 论文机制总览图](assets/figures/overview/star-p-lyamath-multi-agent-reasoning-under-persistent-meta-strategic-supervision-hero.svg)
*图示：候选主图不可靠，已回退为论文核心机制总览 SVG。*

- **当前状态：** llm_failed（LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}）
- **核心技术点：** 本次精读未成功，暂不展示结构化核心点，避免误导。

## 三、候选但未完成深读的论文

- **EngiAI: A Multi-Agent Framework and Benchmark Suite for LLM-Driven Engineering Design**
  - 状态：llm\_failed
  - 原因：LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}
- **RoboJailBench: Benchmarking Adversarial Attacks and Defenses in Embodied Robotic Agents**
  - 状态：llm\_failed
  - 原因：LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}
- **STAR-PólyaMath: Multi-Agent Reasoning under Persistent Meta-Strategic Supervision**
  - 状态：llm\_failed
  - 原因：LLM 分析失败: LLM 调用失败: Error code: 429 - {'error': {'message': 'Budget has been exceeded! Current cost: 401.77440125, Max budget: 400', 'type': 'budget\_exceeded', 'param': None, 'code': '429'}}

## 四、总结

- Agent 研究继续从模型能力转向执行链与系统边界。
- 更值得追踪的是评测、约束、协议和运行时设计。
- 如果把今天的论文连起来看，一个明显变化是：大家越来越少把 Agent 当成单一模型能力问题，而是把它当成执行链、工具层、评测层和安全层共同构成的系统问题。
- 这意味着后续真正有开拓性的研究，往往不是再加一点 prompt 技巧，而是重新定义 Agent 应该如何被评测、如何被约束，以及如何在真实环境里更稳地工作。
