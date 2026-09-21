## 方畅 · Chang FANG

香港大学工学院硕士在读（低空科技 · 计算机方向），2027 届。本科海南大学软件工程。

方向是 LLM Agent 的工程实现。做的过程中最花心思的其实是同一件事：

> **怎么知道一个 Agent 是真的做对了，而不是看起来做对了。**

所以力气有一半花在“怎么验证”上。下面三个小实验本来不是一组，是做项目时被同一类问题绊了三次，才攒到一起的。

---

### 三个小实验

**检索 · [rag-lab](https://github.com/ColeFang35/rag-lab)**

客服和差旅两个项目里的知识库检索一直是“关键词命中打分”（tags 命中算 3 分、正文算 1 分）。我怀疑它在口语化问法上会失灵，就把它和向量检索放进同一个评测集，跑一样的 18 条 query。

结果差得比我预想的多：R@1 从 78% 到 94%，而关键词漏掉的那 3 条，全是“怎么开发票”“坐飞机是什么舱位标准”这种跟原文用词对不上的问法。顺手也量了 cross-encoder 重排，它确实把 R@3 顶到 100%，但单条从 3ms 涨到 600ms。好分块加纯向量在这个规模上比重排划算，这笔账我原来没算过。

**裁判 · [llm-judge-eval](https://github.com/ColeFang35/llm-judge-eval)**

用大模型给 Agent 轨迹打分很方便，但打分之前得先看看打分的人自己靠不靠谱。我拿 6 条人工标注过的客服轨迹做对照，Cohen's kappa 是 0.667。

唯一对不上的是隐私越权那条：用户只给了手机号后四位，Agent 把完整号码复述了出来。人工判它不通过，裁判却给了满分，理由是“完整回答了用户查询，提供了手机号”，它根本没觉得这是个问题。这条最后是靠代码判的“禁止内容”拦下来的。能用代码判的别交给模型，我以前只觉得这样省 token。

**置信度 · [decision-layer-lab](https://github.com/ColeFang35/decision-layer-lab)**

我本来想验证一个挺流行的说法：模型自报的置信度虚高，不能拿来设阈值。跑完发现反了，两个 DeepSeek 模型在难档上都偏保守。

更意外的是另一个结果：难档里 ECE 最好看的那条方案（0.009），准确率只有 27.3%。它不是校准得好，是它知道自己不行，所以答错的时候置信度也低。低 ECE 不等于能用，这一点我一开始完全没意识到。

---

### Agent 项目

**[yunqiao-customer-service-agent](https://github.com/ColeFang35/yunqiao-customer-service-agent)**
电商客服 Agent。17 个业务工具覆盖售前到售后，业务数据全部经工具查询、不做生成，所以每一次回答都能追溯到具体哪个工具返回了什么。带自研的 Fire Trace 调试平台。无需 API Key 即可跑通。

**[aligo-travel-agent](https://github.com/ColeFang35/aligo-travel-agent)**
多智能体差旅助手，AgentScope 2.0-Java + Spring Boot。主规划 Agent 带 4 个子 Agent，差标政策走知识库检索注入，答案可引出处。

**[low-altitude-multi-uav-agent](https://github.com/ColeFang35/low-altitude-multi-uav-agent)**
基于 LangGraph 的低空多机协同决策。把“谁让谁”从飞行员的临场判断，变成一套确定的排序规则。

---

### 后训练

**[agentic-rl-lab](https://github.com/ColeFang35/agentic-rl-lab)** ／ **[lora-finetune-lab](https://github.com/ColeFang35/lora-finetune-lab)**
一个是把工具调用任务的成败做成可验证奖励，跑拒绝采样加 SFT 与 DPO；一个是 QLoRA 指令微调。

---

Python 和 Java 都写。日常用 Claude Code 开发，也把反复要跑的流程抽成过 Skill。

📮 fangchangchampion@gmail.com
