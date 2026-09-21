## 方畅 · Chang FANG

香港大学工学院硕士在读（低空科技 · 计算机方向），2027 届。本科海南大学软件工程。

方向是 LLM Agent 的工程实现。做的过程中最花心思的其实是同一件事：

> **怎么知道一个 Agent 是真的做对了，而不是看起来做对了。**

所以力气有一半花在“怎么验证”上。下面三个实验台是这条线上的三层，每层一个可复现的小实验。

---

### 一、验证这条线：三层信号，各自可不可信

| 层 | 仓库 | 实测结论 |
|---|---|---|
| **检索层** | [rag-lab](https://github.com/ColeFang35/rag-lab) | 向量检索 R@1 **94%**，原关键词方案 78%；关键词漏掉的全是「用词和原文对不上」的口语问法。cross-encoder 重排把 R@3 提到 100%，但延迟从 3ms 涨到 600ms —— 值不值要看场景。ONNX 跑 embedding，不需要 GPU 也不需要 API Key。 |
| **裁判层** | [llm-judge-eval](https://github.com/ColeFang35/llm-judge-eval) | 用 LLM 判 Agent 的执行轨迹，与人工标注的 **Cohen's kappa 0.667**（规则基线那种 1.0 是假象——它是照着人工标注写的）。唯一的分歧暴露了裁判的系统盲区：它把「复述用户完整手机号」当成「回答完整」给了满分，**是程序化硬指标把它拦住的**。纯标准库实现，零依赖。 |
| **决策层** | [decision-layer-lab](https://github.com/ColeFang35/decision-layer-lab) | 模型自报的置信度能不能拿来设阈值（低置信度转人工）？校准确实随难度退化，ECE 从 0.002 走到 0.090。但**最难那一档里 ECE 最好看的一条（0.009）准确率只有 27.3%** —— 它不是校准得好，是「知道自己不行」。 |

> 三个实验指向同一句话：**别信单一信号，要看它和别的信号对不对得上。**
> 检索要看重排的收益配不配得上延迟，评测要看裁判和人工对不对得齐，
> 置信度要和准确率一起看。

---

### 二、Agent 项目

**[yunqiao-customer-service-agent](https://github.com/ColeFang35/yunqiao-customer-service-agent)**
电商客服 Agent。17 个业务工具覆盖售前到售后，业务数据全部经工具查询、不做生成，所以每一次回答都能追溯到具体哪个工具返回了什么。带自研的 Fire Trace 调试平台。无需 API Key 即可跑通。

**[aligo-travel-agent](https://github.com/ColeFang35/aligo-travel-agent)**
多智能体差旅助手，AgentScope 2.0-Java + Spring Boot。主规划 Agent 带 4 个子 Agent，差标政策走知识库检索注入，答案可引出处。

**[low-altitude-multi-uav-agent](https://github.com/ColeFang35/low-altitude-multi-uav-agent)**
基于 LangGraph 的低空多机协同决策。把“谁让谁”从飞行员的临场判断，变成一套确定的排序规则。

---

### 三、后训练的两个小实验台

**[agentic-rl-lab](https://github.com/ColeFang35/agentic-rl-lab)** ／ **[lora-finetune-lab](https://github.com/ColeFang35/lora-finetune-lab)**
一个是把工具调用任务的成败做成可验证奖励，跑拒绝采样加 SFT 与 DPO；一个是 QLoRA 指令微调。

---

Python 和 Java 都写。日常用 Claude Code 开发，也把反复要跑的流程抽成过 Skill。

📮 fangchangchampion@gmail.com
