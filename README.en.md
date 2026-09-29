## Chang FANG · 方畅

[中文](README.md) · **English**

Master's student at the Faculty of Engineering, University of Hong Kong (Low-Altitude Technology, Computer Science track), class of 2027. BEng in Software Engineering, Hainan University.

I work on the engineering side of LLM agents. One question has taken most of my attention:

> **How do you know an agent actually got it right — rather than just looking like it did?**

So roughly half my effort goes into *verification*. The four small labs below weren't planned as a set; I kept running into the same class of problem while building projects, and eventually collected them here.

---

### Four small labs

**Retrieval · [rag-lab](https://github.com/ColeFang35/rag-lab)**

Both my customer-service and travel-agent projects scored knowledge-base retrieval by keyword hits (a tag match counted 3 points, a body match 1). I suspected it would fail on colloquial phrasing, so I put it head-to-head with vector retrieval on the same eval set of 18 queries.

The gap was wider than I expected: R@1 went from 78% to 94%, and all three queries keyword search missed were phrased nothing like the source text — for example asking for "an invoice" in everyday wording rather than the wording the docs use. I also measured cross-encoder reranking: it did push R@3 to 100%, but per-query latency went from 3 ms to 600 ms. At this scale, good chunking plus plain vector search beats reranking — a trade-off I hadn't costed out before.

The same repo holds a second question. Vectorization originally ran on ONNX Runtime, and I wanted to know whether switching to another vendor's inference framework would be faster. The first measurement said "twice as slow, and retrieval drops 5 points." **That conclusion was wrong.** The ONNX model had already been int8-quantized by onnxruntime's own quantizer; OpenVINO was simply running someone else's quantization output, without ever using its own toolchain. Once I re-quantized from FP32 with NNCF and added an FP32 row as an accuracy reference, latency, accuracy and end-to-end metrics all improved together, and the gap narrowed from 2.07× to 1.66×. **When comparing inference frameworks, the quantization toolchain is a variable you have to control first** — otherwise you arrive at an answer that looks entirely reasonable and is quietly biased.

**Judge · [llm-judge-eval](https://github.com/ColeFang35/llm-judge-eval)**

Using a large model to score agent trajectories is convenient, but before trusting the score you should check whether the scorer itself is reliable. I compared it against 6 human-labelled customer-service trajectories; Cohen's kappa came out at 0.667.

The one case that disagreed was a privacy violation: the user had given only the last four digits of a phone number, and the agent read the full number back. The human label said fail; the judge gave full marks, reasoning that it "fully answered the user's query by providing the phone number" — it simply didn't register a problem. That case was ultimately caught by a code-based "forbidden content" check instead. Don't hand to a model what code can decide — I used to think that was only about saving tokens.

**Confidence · [decision-layer-lab](https://github.com/ColeFang35/decision-layer-lab)**

I set out to confirm a popular claim: that a model's self-reported confidence runs high and can't be used to set thresholds. It came out the other way — both DeepSeek models were *under*-confident on the hard tier.

The more surprising result was a different one. The variant with the best-looking ECE on the hard tier (0.009) had only 27.3% accuracy. It wasn't well calibrated; it knew it was out of its depth, so its confidence stayed low even when it was wrong. Low ECE does not mean usable — something I hadn't appreciated at all going in.

**Image consistency · [image-gen-eval-lab](https://github.com/ColeFang35/image-gen-eval-lab)**

Text-to-image work is often acceptance-tested by using CLIP similarity as a proxy for "character consistency." I set out to show it can't actually separate the same character from a near-relative one — **and got it backwards: it separates them quite clearly.** Across 24 images (3 character prompts × 8 seeds), same-character pairs averaged 0.912, near-relatives differing only in hair colour averaged 0.867, and unrelated pairs 0.418 — AUC 0.920.

But an AUC of 0.920 still doesn't make it usable. One layer down: at the best threshold (0.897) accuracy is only 0.870 — 4 of 28 same-character pairs missed, 8 of 64 near-relative pairs falsely flagged — and **the lowest same-character score (0.872) sits below the highest near-relative one (0.925)**. The two distributions overlap. The verdict is **fine as a coarse filter, not as an acceptance test**: it can cut 500 candidate images down to 50, but a human still has to look at those 50.

The lesson is that a metric has to be asked about at three levels — **can it separate (AUC) / by how much (overlap) / how wrong at the threshold**. Looking good at the first level says nothing about the third.

---

### Agent projects

**[yunqiao-customer-service-agent](https://github.com/ColeFang35/yunqiao-customer-service-agent)**
E-commerce customer-service agent. 17 business tools spanning pre-sales through after-sales; all business data comes from tool calls and is never generated, so every answer traces back to which tool returned what. Ships with a purpose-built Fire Trace debugging platform. Runs end to end with no API key.

**[aligo-travel-agent](https://github.com/ColeFang35/aligo-travel-agent)**
Multi-agent travel assistant built on AgentScope 2.0-Java + Spring Boot. A lead planning agent with 4 sub-agents; travel-policy questions go through knowledge-base retrieval injected into the prompt, so answers can cite their source.

**[low-altitude-multi-uav-agent](https://github.com/ColeFang35/low-altitude-multi-uav-agent)**
Low-altitude multi-UAV coordination built on LangGraph. Turns "who yields to whom" from a pilot's in-the-moment judgement into a deterministic ordering rule.

---

### Post-training

**[agentic-rl-lab](https://github.com/ColeFang35/agentic-rl-lab)** / **[lora-finetune-lab](https://github.com/ColeFang35/lora-finetune-lab)**
One turns tool-calling success and failure into a verifiable reward and runs rejection sampling plus SFT and DPO; the other is QLoRA instruction tuning.

---

I write both Python and Java. I use Claude Code as my main development tool day to day, and have packaged repeated workflows into reusable Skills.

📮 fangchangchampion@gmail.com
