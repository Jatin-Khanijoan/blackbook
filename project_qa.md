# Project Q&A — End-to-End Autonomous System

## 1. What is the problem statement?

Existing AI models used in high-stakes domains such as finance, medicine, law, and science suffer from three critical limitations when making decisions:

1. **Overconfidence and uncalibrated uncertainty:** Models often produce highly confident outputs even when underlying information is incomplete, ambiguous, or outdated.
2. **Retrieval-decision misalignment:** Traditional retrieval-augmented generation (RAG) systems rank documents purely by semantic similarity or BM25 scores, which is entirely decoupled from the actual downstream goal of predictive accuracy under calibrated uncertainty.
3. **Multi-agent credit assignment bottleneck:** In multi-agent settings (such as institutional portfolio management where different desks handle different sectors), assigning precise blame or credit to individual agents when a global performance metric underperforms is mathematically intractable using conventional scalar reward distributions.

Human decision-making also suffers from cognitive biases (loss aversion, herding, mental accounting), limited analytical capacity, and inability to simultaneously evaluate thousands of scenarios.

## 2. What is the main goal of this project?

The primary goal is to design and implement an **end-to-end autonomous decision-making framework** capable of generating calibrated, structured recommendations for complex real-world problems. Specifically, the system aims to:

- **Identify** the current problem state using Markov models
- **Retrieve** the most critical information sources by minimizing downstream conformal prediction set volumes (not just semantic similarity)
- **Map** those resources to possible decisions through coordination graphs
- **Evaluate** outcomes via factored optimal transport dynamics with distribution-free uncertainty guarantees

The framework targets multiple domains: **Finance, Medical, Science, and Law**, establishing a general, mathematically rigorous architecture for autonomous decision support.

## 3. What inspired this idea? / What is the research background?

The project draws inspiration from several converging research threads:

- **Retrieval-Augmented Generation (RAG)** systems showed that grounding LLMs in external evidence reduces hallucination, but their retrieval criteria (cosine similarity) don't align with decision quality.
- **Multi-Agent Reinforcement Learning (MARL)** research demonstrated that specialized collaborative agents can outperform single-agent systems, yet credit assignment remains unresolved.
- **Conformal Prediction** theory provides finite-sample coverage guarantees without distributional assumptions, offering a rigorous alternative to Bayesian uncertainty.
- **Optimal Transport** theory (Wasserstein distances, Sinkhorn algorithm) enables principled comparison of probability distributions for reward shaping.
- **Behavioural finance** research (Kahneman & Tversky's Prospect Theory, Thaler's Mental Accounting) documented systematic human biases that algorithmic systems can eliminate.

The team reviewed **110 research papers** spanning AI in finance, deep reinforcement learning, sentiment analysis, multi-agent systems, conformal prediction, and optimal transport to build the theoretical foundation.

## 4. Why this idea?

This idea addresses a genuine gap in the current AI landscape:

- **SDG 8 (Decent Work and Economic Growth):** Automating decision-making enhances economic productivity, creates quality employment in financial technology sectors, and enables better-informed investment decisions that drive sustainable growth.
- **SDG 9 (Industry, Innovation and Infrastructure):** Developing innovative AI infrastructure enhances industrial competitiveness and builds resilient technological systems across finance, healthcare, and legal services.
- The project specifically targets **Indian equity markets** (NSE, BSE), where documented problems include FII volatility, sectoral heterogeneity, regulatory constraints (SEBI/RBI), and widespread behavioural biases among retail investors.
- Existing solutions are either too narrow (single-domain), too shallow (cosine-similarity retrieval), or lack uncertainty quantification entirely.

## 5. What is the proposed solution? (High-level architecture)

The framework integrates two core modules:

### Module 1: IDCR — Information-Directed Conformal Retrieval
Instead of retrieving documents by semantic similarity, IDCR:
- Models document retrieval as **submodular optimization** for continuous uncertainty reduction
- Uses **conformal prediction sets** to quantify distribution-free uncertainty
- Selects documents that minimize downstream prediction set volumes (i.e., maximally reduce uncertainty)
- Employs Bayesian posterior updates with precision matrices derived from financial domain encoders (FinBERT)

### Module 2: FOTRS — Factored Optimal Transport Reward Shaping
To solve multi-agent coordination:
- Decomposes global portfolio objectives into **decentralized local rewards** over coordination graphs
- Uses **entropy-regularized Sinkhorn distances** for computationally efficient optimal transport
- Ensures policy alignment toward **Nash equilibria** through potential game formulations
- Employs sector-specific agents (inspired by Indian market sectoral heterogeneity) with dynamic edge reweighting

## 6. What was the expected output?

The expected outputs include:

1. **A unified autonomous decision-making framework** integrating IDCR and FOTRS modules
2. **Calibrated, verifiable recommendations** with distribution-free uncertainty guarantees (conformal prediction sets)
3. **Multi-agent portfolio allocation system** for Indian equity markets demonstrating:
   - Superior risk-adjusted returns (Sharpe ratio benchmarks against existing studies)
   - Reduced catastrophic tail risk through Mahalanobis-aware transport costs
   - Elimination of emotional/biased trading decisions
4. **Cross-domain applicability** demonstrated across finance, medicine, science, and law
5. **Auditable, explainable decisions** using SHAP and LIME for regulatory compliance under SEBI/RBI guidelines

## 7. What domains does the system apply to?

While extensively grounded in **financial portfolio management** (Indian equity markets: NSE, BSE, NIFTY 50), the framework is designed as a **general decision-making architecture** applicable to:

- **Finance:** Algorithmic trading, portfolio allocation, robo-advisory, risk management
- **Medicine:** Diagnostic decision support, clinical guideline retrieval, treatment planning
- **Science:** Research literature synthesis, experimental design, hypothesis evaluation
- **Law:** Legal interpretation, regulation compliance, case law retrieval

All these domains share the same fundamental four-stage decision structure: state understanding → information gathering → decision mapping → outcome evaluation.

## 8. What are the key technologies and frameworks used?

| Category | Technologies |
|----------|-------------|
| **Retrieval & NLP** | RAG, FinBERT, Sentence-BERT, FAISS, Transformers, LoRA fine-tuning |
| **Uncertainty Quantification** | Conformal Prediction, Bayesian posterior updates, submodular optimization |
| **Reinforcement Learning** | PPO, SAC, TD3, DQN, MADDPG, QMIX, MARL |
| **Optimal Transport** | Wasserstein distances, Sinkhorn algorithm, entropy regularization |
| **Time-Series** | LSTM, BiLSTM, CNN-LSTM, GARCH, DCC-GARCH, Informer, PatchTST |
| **Explainability** | SHAP, LIME |
| **Simulation** | FinRL-Meta environments |

## 9. What are the key contributions?

1. **Retrieval-decision alignment:** First framework to replace semantic-similarity retrieval with information-gain-directed document selection using conformal set volume minimization.
2. **Principled multi-agent credit assignment:** FOTRS resolves the MARL credit assignment bottleneck via optimal transport-based reward factorization over coordination graphs.
3. **Distribution-free uncertainty guarantees:** Unlike Bayesian credible intervals, conformal prediction sets provide finite-sample coverage without distributional assumptions.
4. **Sector-aware agent topology:** Inspired by Indian market studies, the coordination graph partitions agents by sector with dynamic edge reweighting based on macroeconomic regimes.
5. **Bias elimination:** The system implicitly removes prospect theory biases (loss aversion, mental accounting) by replacing human judgment with calibrated algorithmic allocation.

## 10. What challenges does the system address?

- **Dynamic knowledge:** Financial markets, medical guidelines, and legal regulations change continuously; the framework actively retrieves the most current evidence.
- **Information overload:** Human analysts cannot process thousands of documents; IDCR selects only those that maximally reduce predictive uncertainty.
- **Multi-agent coordination:** In institutional portfolios, global metrics (Sharpe ratio) obscure individual agent contributions; FOTRS factorizes rewards mathematically.
- **Regulatory compliance:** SEBI and RBI require transparency and auditability; conformal sets and SHAP explanations provide auditable uncertainty quantification.
- **Behavioural biases:** Human investors exhibit loss aversion, herding, and panic selling; algorithmic systems eliminate these emotional distortions.

## 11. Who is this project by?

| Name | Roll Number |
|------|-------------|
| Manas Nanivadekar | 22102033 |
| Jatin Khanijoan | 22102094 |
| Swayam Kothekar | 22102047 |
| Amaan Khan | 22102007 |

**Guide:** Prof. Deepak S. Khachane  
**Institution:** A. P. Shah Institute of Technology, Thane (University of Mumbai)  
**Academic Year:** 2025–2026  
**Degree:** Bachelor of Computer Engineering

## 12. What is the significance and impact?

- **For financial markets:** Improves algorithmic portfolio allocation, mitigates catastrophic risk, and reduces emotionally driven trades (studies show AI rebalancing reduces emotional trades by ~90%).
- **For regulatory bodies:** Provides auditable, distribution-free uncertainty guarantees that satisfy SEBI/RBI transparency requirements.
- **For developing economies:** Aligns with India's push toward digital financial inclusion and algorithmic trading infrastructure.
- **For AI research:** Establishes a template for retrieval-decision aligned systems that can generalize across high-stakes domains beyond finance.

## 13. Keywords

Artificial Intelligence, Information Retrieval, Conformal Prediction, Multi-Agent Systems, Optimal Transport, Reinforcement Learning, Portfolio Management, Uncertainty Quantification, Algorithmic Trading, Explainable AI.

---

*This document was generated based on the content of the main.tex project report for "End-to-End Autonomous System" (B.E. Computer Engineering, 2025–2026).*
