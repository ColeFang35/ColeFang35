## 方畅 · Chang FANG

[English](README.en.md) · **中文**

香港大学工学院硕士在读（低空科技 · 计算机方向），2027 届。本科海南大学软件工程。

方向是 LLM Agent 的工程实现。做的过程中最花心思的其实是同一件事：

> **怎么知道一个 Agent 是真的做对了，而不是看起来做对了。**

所以力气有一半花在“怎么验证”上。下面四个小实验本来不是一组，是做项目时被同一类问题绊了四次，才攒到一起的。

---

### 四个小实验

**检索 · [rag-lab](https://github.com/ColeFang35/rag-lab)**

客服和差旅两个项目里的知识库检索一直是“关键词命中打分”（tags 命中算 3 分、正文算 1 分）。我怀疑它在口语化问法上会失灵，就把它和向量检索放进同一个评测集，跑一样的 18 条 query。

结果差得比我预想的多：R@1 从 78% 到 94%，而关键词漏掉的那 3 条，全是“怎么开发票”“坐飞机是什么舱位标准”这种跟原文用词对不上的问法。顺手也量了 cross-encoder 重排，它确实把 R@3 顶到 100%，但单条从 3ms 涨到 600ms。好分块加纯向量在这个规模上比重排划算，这笔账我原来没算过。

同一个仓库里我还追了第二个问题。向量化原本跑在 ONNX Runtime 上，我想知道换一个厂商的推理框架会不会更快——第一版量出来是“慢一倍，检索指标还掉 5 个点”。**但这个结论是错的。** 那个 ONNX 模型本来就是 onnxruntime 自家 quantizer 压出来的 int8，OpenVINO 只是照跑别人的量化产物，没用上自己的工具链。从 FP32 用 NNCF 重新量化、再加一档 FP32 当精度参照之后，延迟、精度、端到端指标三件事一起变好，差距从 2.07 倍缩到 1.66 倍。**做框架对比，量化工具链是必须先控制的变量**——不控制，就会得出一个看起来很有道理、其实有偏的结论。

**裁判 · [llm-judge-eval](https://github.com/ColeFang35/llm-judge-eval)**

用大模型给 Agent 轨迹打分很方便，但打分之前得先看看打分的人自己靠不靠谱。我拿 6 条人工标注过的客服轨迹做对照，Cohen's kappa 是 0.667。

唯一对不上的是隐私越权那条：用户只给了手机号后四位，Agent 把完整号码复述了出来。人工判它不通过，裁判却给了满分，理由是“完整回答了用户查询，提供了手机号”，它根本没觉得这是个问题。这条最后是靠代码判的“禁止内容”拦下来的。能用代码判的别交给模型，我以前只觉得这样省 token。

**置信度 · [decision-layer-lab](https://github.com/ColeFang35/decision-layer-lab)**

我本来想验证一个挺流行的说法：模型自报的置信度虚高，不能拿来设阈值。跑完发现反了，两个 DeepSeek 模型在难档上都偏保守。

更意外的是另一个结果：难档里 ECE 最好看的那条方案（0.009），准确率只有 27.3%。它不是校准得好，是它知道自己不行，所以答错的时候置信度也低。低 ECE 不等于能用，这一点我一开始完全没意识到。

**图像一致性 · [image-gen-eval-lab](https://github.com/ColeFang35/image-gen-eval-lab)**

文生图有个常见的验收做法：拿 CLIP 相似度当“角色一致性”的指标。我想验证它其实分不开同一个角色和同类不同角色——**结果押反了，它分得挺开**：3 个角色 × 8 个 seed 共 24 张图，同角色对均值 0.912，只改了发色的近亲角色对 0.867，无关的 0.418，AUC 0.920。

但 0.920 也不能直接说能用。再往下拆一层：最佳阈值 0.897 处准确率只有 0.870，同角色 28 对漏了 4 对，近亲 64 对误报 8 对，而且**同角色最低的 0.872 比近亲最高的 0.925 还低**——两组分布是交叠的。结论是**能当粗筛，不能当验收**：它能把要看的图从 500 张缩到 50 张，那 50 张还得人看。

这件事教会我指标要分三层问：**能不能分（AUC）／分开多少（分布重叠）／阈值处错多少**。第一层好看，不代表第三层能用。

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
