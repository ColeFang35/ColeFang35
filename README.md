## 方畅 · Chang FANG

香港大学工学院硕士在读（低空科技 · 计算机方向），2027 届。本科海南大学软件工程。

方向是 LLM Agent 的工程实现。做的过程中最花心思的其实是同一件事：

> **怎么知道一个 Agent 是真的做对了，而不是看起来做对了。**

所以几个项目里有一半的力气花在「怎么验证」上：工具调用能不能追溯到真实数据、评测的裁判自己可不可靠、一条规则到底生效没生效。

---

### 项目

**[yunqiao-customer-service-agent](https://github.com/ColeFang35/yunqiao-customer-service-agent)**
电商客服 Agent。17 个业务工具覆盖售前到售后，业务数据全部经工具查询、不做生成，所以每一次回答都能追溯到具体哪个工具返回了什么。带自研的 Fire Trace 调试平台。无需 API Key 即可跑通。

**[llm-judge-eval](https://github.com/ColeFang35/llm-judge-eval)**
面向 Agent 任务的评测框架。能程序精确判定的用代码判，只有开放式质量才交给模型打分；而且裁判本身也要先证明可信（Cohen's kappa、自一致性、位置偏见）。纯标准库实现，零依赖。

**[aligo-travel-agent](https://github.com/ColeFang35/aligo-travel-agent)**
多智能体差旅助手，AgentScope 2.0-Java + Spring Boot。主规划 Agent 带 4 个子 Agent，差标政策走知识库检索注入，答案可引出处。

**[low-altitude-multi-uav-agent](https://github.com/ColeFang35/low-altitude-multi-uav-agent)**
基于 LangGraph 的低空多机协同决策。把「谁让谁」从飞行员的临场判断，变成一套确定的排序规则。

**[agentic-rl-lab](https://github.com/ColeFang35/agentic-rl-lab)** ／ **[lora-finetune-lab](https://github.com/ColeFang35/lora-finetune-lab)**
后训练的两个小实验台。一个是把工具调用任务的成败做成可验证奖励，跑拒绝采样加 SFT 与 DPO；一个是 QLoRA 指令微调。

---

Python 和 Java 都写。日常用 Claude Code 开发，也把反复要跑的流程抽成过 Skill。

📮 fangchangchampion@gmail.com
