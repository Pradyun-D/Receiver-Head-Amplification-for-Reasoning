# 🧠 Receiver Head Amplification for Reasoning: A Mechanistic Study

## 📝 Introduction
This project investigates a **"white-box"** approach to improving the reasoning capabilities of Large Language Models (LLMs). Specifically, it explores whether amplifying **receiver heads**—attention heads that focus on critical **"Thought Anchors"** such as planning, backtracking, and step termination—can improve success rates on long-horizon reasoning tasks.

Using **Deepseek-R1-Distill-Qwen-1.5b** as the experimental base, the hypothesis was tested manually by scaling these attention signals to induce more robust reasoning patterns without the need for reinforcement learning (RL) or complex reward modeling.

---

## 🛠 Methodology

### 1️⃣ Dataset Selection
Experiments were conducted using the same dataset used in the **"Thought Anchors"** paper.
* **Source:** 20 curated problems from the **MATH dataset** (Hendrycks et al., 2020).
* **Criteria:** Problems were selected based on a baseline solution rate of **25-75%** to ensure tasks were challenging but within the model's fundamental capability.

### 2️⃣ Identifying Receiver Heads
Following the "Thought Anchors" framework, receiver heads were identified by calculating the **kurtosis** of vertical attention patterns across the model's layers. 

* **The Logic:** High kurtosis indicates **"spiky"** attention, where a head ignores the majority of the context to focus intensely on specific anchor points.
* **Primary Heads:** **Layer 21 (Head 5 and Head 2)** were identified as the most prominent receiver heads.

<img width="687" height="688" alt="Kurtosis Distribution Graph" src="https://github.com/user-attachments/assets/56741c0f-9f23-4415-9880-c9e5c6e24fa5" />

### 3️⃣ Amplification Setup
**Pre-forward hooks** were used to intercept and scale the attention outputs (the input to the O-projection) during inference.
* **Scaling Factors:** `1.2x`, `1.5x`, `5x`, and `10x`.
* **Inference Config:** Top-p sampling with a max token limit of **4,096** to capture full reasoning traces.

---

## 📊 Results & Visuals

The study found that static amplification generally **degraded** reasoning performance. As the scaling factor increased, the model’s ability to maintain logical coherence and mathematical accuracy declined.

### 📉 Performance Breakdown
| Scaling Factor | Success Rate (%) | Primary Observation |
| :--- | :--- | :--- |
| **Baseline (1.0x)** | **~25%** | Standard CoT reasoning. |
| **1.2x Scaling** | **5.3%** | Significant drop; reasoning begins to fray. |
| **1.5x Scaling** | **0%** | Complete failure; model loses coherence. |
| **5x - 10x Scaling** | **0%** | Output degrades into repetitive loops or gibberish. |

---

## ⚠️ Primary Failure Modes

### 🔄 A. Backtracking Loops
Amplifying receiver heads often trapped the model in "backtracking loops." Because the model was over-attending to a backtracking anchor, it would repeatedly perform the same calculation step without recognizing that the task was complete.

<img width="2784" height="1536" alt="Backtracking Loop Example" src="https://github.com/user-attachments/assets/c18832f7-c56e-4228-8d0d-938fae42b67d" />

### 🧮 B. Computation & Arithmetic Breakdown
Even at the lowest scaling factor (`1.2x`), the model began failing at basic arithmetic that it solved correctly in the baseline. This suggests that manual scaling disrupts the precision of the model's underlying numerical representations.

<img width="2816" height="1536" alt="Arithmetic Breakdown" src="https://github.com/user-attachments/assets/b1f0a9ba-346f-41e0-a84c-c829817b444c" />

### 📍 C. Plan Fixation
Stronger planning signals resulted in **"Plan Fixation,"** where the model became unable to pivot. Even when the model explicitly noted that its current approach was failing, the amplified signal forced it to continue the flawed strategy.

<img width="2816" height="1536" alt="Plan Fixation Example" src="https://github.com/user-attachments/assets/2876241e-b6c6-432c-9349-7eb798d0e2dc" />

---

## 💡 Conclusion
Static amplification of receiver heads is insufficient for enhancing reasoning performance. While these heads are critical for handling "Thought Anchors," boosting them linearly disrupts the delicate balance required for multi-step logic. 

**🚀 Future Work:** This research suggests that a **dynamic amplification** method—potentially using a classifier to boost specific anchors only when contextually relevant—may be more effective than static scaling.

---

> ℹ️ **For more detailed information, technical analysis, and extended data, see the full project document:** > [Project Documentation](https://docs.google.com/document/d/1UQjBcAoQXHl7rdWCyTFYGyXSTBupzp68ljiSLdioK64/edit?usp=sharing)

---
*Inspired by the research framework in: [Thought Anchors: Which LLM Reasoning Steps Matter?](https://arxiv.org/abs/2506.19143)*
