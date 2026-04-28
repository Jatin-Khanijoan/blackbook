# Viva Q&A — End-to-End Autonomous System
*Evaluator-style questions with clear, simple answers.*

---

## Q1. What exactly is your project in one simple sentence?

**Answer:** It is an AI system that makes reliable, uncertainty-aware decisions in high-stakes fields like finance, medicine, and law by retrieving only the most useful information and coordinating multiple specialized agents with mathematical guarantees.

Think of it as a smart research team where each member is an expert in one sector (like Banking or Pharma), the team librarian only fetches documents that actually reduce confusion (not just similar keywords), and every recommendation comes with a provable confidence range.

---

## Q2. Why should anyone use your system instead of ChatGPT or a regular trading app?

**Answer:** Because ChatGPT and standard apps give you **one answer** with no honest measure of how wrong they might be.

- **ChatGPT** can sound extremely confident while being completely outdated or hallucinating facts. It has no built-in way to say *"I am 90% sure the answer lies between X and Y."*
- **Regular trading apps** use static rules or simple similarity search. They retrieve articles that *look* like your query, not articles that actually help the decision.

Our system outputs a **prediction set** (a bounded range of safe decisions) backed by a mathematical guarantee: if you set 90% confidence, the true answer is inside that set at least 90% of the time—no matter what the market distribution looks like.

---

## Q3. How is your approach more reliable than traditional methods?

**Answer:** Traditional methods suffer from three reliability problems we fix:

| Problem | Traditional Approach | Our Approach |
|---------|---------------------|--------------|
| **Overconfidence** | Gives a single number (e.g., "invest 50%") | Gives a calibrated set (e.g., "invest 45–55% with 90% guarantee") |
| **Retrieval mismatch** | Fetches documents by keyword/cosine similarity | Fetches documents by how much they reduce decision uncertainty |
| **Multi-agent blame game** | One giant model or uncoordinated agents | Specialized agents with mathematically factored rewards so everyone knows their exact contribution |

In plain terms: traditional RAG is like a librarian who gives you 10 books with similar titles, even if they all say the same thing. IDCR is like a librarian who gives you the exact 5 papers that remove your doubts.

---

## Q4. What is Conformal Prediction, and why did researchers develop it?

**Answer:** Conformal Prediction (CP) is a statistical framework that wraps around *any* machine learning model to add honest uncertainty intervals.

- **Origin:** The theory was developed by statisticians **Vladimir Vovk, Alex Gammerman, and Glenn Shafer** around 2005, and later made practical for modern ML by researchers like **Angelopoulos and Bates** (Stanford/UC Berkeley, 2021–2023).
- **Why it was needed:** Classical ML models output probabilities that are often miscalibrated. A model might say "95% confident" but actually be wrong 30% of the time. CP fixes this by using a small held-out **calibration set** to learn the real error rate and construct sets that are guaranteed to contain the true answer.
- **The key promise:** **Distribution-free coverage.** This means the guarantee holds even if the stock market behaves like a wild bull run, a crash, or anything in between. You do not need to assume returns are normal or stable.

**Real-world analogy:** If a weather app says "30°C" with no error bar, you cannot plan properly. CP gives you "28–32°C with 90% confidence" by checking past forecast errors on a calibration dataset.

---

## Q5. What is the difference between standard RAG and your IDCR module?

**Answer:**

- **Standard RAG (Retrieval-Augmented Generation):** Embeds your query and documents into vectors, then picks the top-k documents with the highest **cosine similarity** (geometric closeness in vector space). This is like Googling a topic and opening the first 5 links because their titles match your keywords.
  - *Flaw:* Those 5 links might all say the same thing. The system learns nothing new, stays overconfident, and the prediction ellipse stays huge.

- **IDCR (Information-Directed Conformal Retrieval):** Picks documents based on how much they **shrink the volume of the conformal prediction ellipsoid**. It uses a greedy submodular algorithm to maximize information gain (measured by log-determinant of the covariance matrix).
  - *Result:* Instead of redundant articles, you get diverse, uncertainty-reducing evidence. In our tests, this compressed the prediction set volume by **5.8×** compared to cosine RAG.

**Simple analogy:** Standard RAG brings you 10 news articles about "Budget 2025" that all repeat the same headline. IDCR brings you the one SEBI circular, one RBI policy note, and one sectoral earnings report that together remove your actual confusion about whether to buy banking stocks.

---

## Q6. Why do you need multiple agents? Why not one big AI model?

**Answer:** One big model suffers from the **curse of dimensionality** and the **credit assignment problem**.

- In an institutional portfolio, different desks manage different sectors (Banking, IT, Pharma, etc.). Each has local expertise and local data.
- If one giant model makes all decisions, it cannot learn which sector decision caused a loss. It is like a group project where everyone gets the same grade—you never know who actually did the work.
- Our system uses **8 specialized sector agents**. Each agent sees only its sector’s information, proposes a local allocation, and the FOTRS coordinator merges them.

**The payoff:** When the global portfolio underperforms, FOTRS tells us exactly which sector agents misaligned, because it computes pairwise Wasserstein distances on a coordination graph.

---

## Q7. What is the credit assignment problem, and how does FOTRS solve it?

**Answer:**

- **The problem:** In multi-agent systems, all agents usually receive the **same global reward** (e.g., total portfolio return). If the portfolio loses money, every agent gets punished equally, even if only the Energy agent made a bad call. This creates free-riding and poor learning.

- **FOTRS solution:** It replaces the single scalar reward with **factored optimal transport rewards**.
  1. It builds a **coordination graph** where edges connect related sectors (e.g., Energy and Metals are linked because their prices move together).
  2. It computes **Wasserstein distances** (Earth Mover’s Distance) between each agent’s actual behavior and the ideal target behavior, edge by edge.
  3. Each agent is penalized only for its own edges. The Banking agent is not blamed for Metal sector volatility.

**Analogy:** Instead of giving the whole class one collective mark, FOTRS grades each student on their specific contribution to the group assignment.

---

## Q8. What were your actual results? Show us the numbers.

**Answer:** On a synthetic NSE-calibrated dataset (8 sectors, 180 months, bull/bear/crisis/recovery regimes), our system was compared against 6 baselines:

| System | Sharpe Ratio | Max Drawdown | Coverage (α=0.10) | Convergence |
|--------|-------------|--------------|-------------------|-------------|
| Zero-Shot LLM | 0.31 | 58.4% | — | — |
| Tool-Augmented LLM | 0.44 | 52.1% | — | — |
| Standard Cosine RAG | 0.51 | 48.6% | 86.0% (fails) | — |
| MARL-QMIX | 0.64 | 43.2% | — | 210 episodes |
| MARL-MAPPO | 0.73 | 41.0% | — | 185 episodes |
| Risk Parity | 0.56 | 67.3% | — | — |
| **IDCR + FOTRS (Ours)** | **0.89** | **34.0%** | **94.4%** | **78 episodes** |

**Key takeaways:**
- **Sharpe Ratio 0.89:** A 22% improvement over the best baseline (MAPPO at 0.73).
- **Max Drawdown 34%:** During crisis regimes, Risk Parity crashed to 67% drawdown. Our system held losses to 34%.
- **Conformal Coverage 94.4%:** We targeted 90% coverage and achieved 94.4%, proving the distribution-free guarantee works in practice.
- **5.8× Volume Reduction:** IDCR shrinked the uncertainty ellipsoid nearly 6× smaller than standard cosine RAG.
- **2.8× Faster Convergence:** FOTRS converged in 78 training episodes vs. MAPPO’s 185.

---

## Q9. How does this system get better results? What is the actual mechanism?

**Answer:** Three mechanisms work together:

1. **Better information (IDCR):** By retrieving documents that maximally reduce covariance volume, the system makes decisions based on orthogonal, high-value evidence rather than repetitive news. This is mathematically guaranteed to be near-optimal (within 63% of the best possible subset) thanks to the submodular greedy bound.

2. **Better coordination (FOTRS):** By routing rewards through optimal transport distances over a sparse coordination graph, each agent learns exactly what it controls. This prevents the "blame everyone" problem and leads to faster, stable convergence.

3. **Honest uncertainty (Conformal Prediction):** The system knows when it does not know. In ambiguous regimes, the prediction set widens, telling the user to be cautious. In clear regimes, it tightens, allowing aggressive confident bets.

Together, these mean the system avoids bad trades caused by redundant data, learns faster because credit is clear, and never pretends to be sure when it is not.

---

## Q10. What if the market suddenly crashes? Will your system break?

**Answer:** No—this is exactly where the system is designed to excel.

- The conformal prediction module does not assume market stability. It uses a **calibration set** that includes crisis periods. When volatility spikes, the prediction sets naturally widen, warning the investor that uncertainty is high.
- The FOTRS coordination graph is built from a **graphical lasso precision matrix**, which captures conditional dependencies between sectors. During crises, when correlations spike (e.g., all sectors fall together), the graph reweights edges dynamically so agents do not independently sell into a panic.
- In our tests, during simulated crisis regimes, Risk Parity lost 67% from peak. Our system lost only 34% because the multi-agent coordination prevented catastrophic joint-policy collapse.

---

## Q11. Why use Optimal Transport (Wasserstein distance)? Why not simple correlation or Euclidean distance?

**Answer:**

- **Euclidean/correlation measures** only compare point-to-point distances or linear relationships. They ignore the full shape of the probability distribution.
- **Wasserstein distance** (Optimal Transport) measures the minimum "work" required to reshape one probability distribution into another. It respects the geometry of the data and handles heavy-tailed, non-Gaussian financial returns far better.
- In FOTRS, we use **entropy-regularized Sinkhorn** iterations to compute this efficiently. This lets us compare entire occupancy distributions (how agents act across many states) rather than just average returns.

**Analogy:** Euclidean distance asks "how far apart are the average test scores of two students?" Wasserstein asks "how much effort is needed to turn one student’s full grade distribution into the other’s?" The second question captures the entire picture.

---

## Q12. Is this only for finance, or can it work elsewhere?

**Answer:** The core architecture is domain-agnostic. We tested finance extensively, but the same pipeline applies to:

- **Medicine:** Retrieve clinical abstracts that reduce diagnostic uncertainty, with multiple agents representing different specializations (cardiology, radiology, etc.).
- **Law:** Retrieve case law and statutes that minimize uncertainty about a legal outcome, with agents for different domains of law.
- **Science:** Fuse literature from multiple disciplines to guide experimental design.

All these domains share the same four-stage structure: understand the state → gather evidence → map to decisions → evaluate outcomes. We validated the cross-domain capability on the MIMIC-IV clinical corpus and GoEmotions NLP tasks.

---

## Q13. How fast is the system? Can it run in real time?

**Answer:**

- **IDCR retrieval latency:** ~17 ms per query for greedy selection. If synergy detection triggers a 2-step lookahead, it remains under 500 ms.
- **FOTRS training convergence:** 78 episodes (vs. 185 for MAPPO), meaning it learns the coordination policy roughly 2.8× faster.
- **Inference:** Once trained, the multi-agent policy produces allocations in milliseconds.

This is fast enough for **daily or weekly portfolio rebalancing** and institutional decision support. It is not designed for high-frequency trading (microsecond scale), but future work with Neural SDEs could extend it there.

---

## Q14. What are the limitations of your system?

**Answer:** Honest limitations include:

1. **Corpus dependency:** If the document knowledge base lacks coverage of a rare event (e.g., a sudden geopolitical shock), the retrieval module cannot invent missing information.
2. **Calibration freshness:** The conformal calibration set must be representative. If market regimes shift dramatically and the calibration set is stale, coverage might temporarily drift until recalibration.
3. **Sector decomposition assumption:** FOTRS assumes the problem can be split into meaningful sectors/agents. For problems without natural partitions, the coordination graph may be less effective.
4. **Latency ceiling:** ~500 ms retrieval is fine for institutional use, but not for ultra-high-frequency trading.

---

## Q15. How do you explain your system's decisions to a regulator or a non-technical investor?

**Answer:** The system provides two layers of explainability:

1. **Conformal prediction sets:** Instead of a black-box number like "invest 50%," the system says: *"Based on your profile and current evidence, the safe allocation to Banking is between 12% and 18%, with a mathematically guaranteed 90% confidence."* This is auditable and requires no trust in the model’s internals.

2. **SHAP and LIME integration:** For every retrieval and allocation decision, we compute SHAP values that show which documents and which market features drove the recommendation. A regulator can audit why the system suggested reducing Pharma exposure on a given day.

This satisfies SEBI and RBI guidelines on AI transparency, which demand that deployed financial models be explainable and auditable.

---

## Q16. Why did you choose submodular optimization? Why not a neural network to pick documents?

**Answer:** We do use neural networks to embed documents, but for **selecting** the subset, submodular optimization is crucial because:

- It provides a **provable guarantee:** The greedy algorithm achieves at least **63%** (1 − 1/e) of the best possible uncertainty reduction, and it does so in linear time.
- A neural network selector would require massive labeled training data (every possible document subset ranked by quality), which is impossible to collect.
- Submodularity naturally models **diminishing returns:** the first document you read teaches you a lot; the tenth similar document teaches you almost nothing. This mirrors how humans research a topic.

---

## Q17. What is the "prediction ellipsoid," and why does shrinking it matter?

**Answer:**

When deciding a portfolio across 8 sectors, a single point prediction (e.g., one weight vector) is dangerous. Instead, our conformal predictor outputs an **ellipsoid** in 8-dimensional space containing all plausible allocations.

- **Volume = Uncertainty:** A large ellipsoid means the system is confused—many allocations look equally valid. A small ellipsoid means the evidence strongly favors a narrow range of decisions.
- **IDCR’s goal:** Pick documents that shrink this volume. Our results show a **5.8× reduction** in volume versus cosine similarity retrieval.

**Analogy:** If you ask "what is the best route to the airport?" a huge ellipsoid means "could be 30 minutes or 2 hours." A tiny ellipsoid means "definitely 35–40 minutes via this road." Smaller is better for decision-making.

---

## Q18. How does this align with Sustainable Development Goals (SDGs)?

**Answer:**

- **SDG 8 (Decent Work and Economic Growth):** Better algorithmic decisions reduce catastrophic losses, improve portfolio efficiency, and create quality employment in financial technology sectors.
- **SDG 9 (Industry, Innovation, and Infrastructure):** The framework builds resilient AI infrastructure. By replacing brittle black-box models with mathematically guaranteed systems, it strengthens the technological backbone of financial institutions.

---

## Q19. What is your publication status?

**Answer:** We have an accepted paper titled **"FOTRS: Factored Optimal Transport Reward Shaping"** at the **2026 International Conference on Computing Theory and Wireless Communications (ICCTWC)**, scheduled for April 2026.

---

## Q20. Summarize the entire project in 30 seconds for a layperson.

**Answer:**

> "Current AI gives you one answer and hopes it is right. Our system gives you a guaranteed safe range of answers and only reads the documents that actually remove your doubts. It also works like a team of 8 specialist analysts who each know exactly how much they contributed to the final decision. In tests on Indian stock market data, it made 22% better risk-adjusted returns, cut crash losses in half, and learned 2.8× faster than existing multi-agent methods."

---

*Prepared for B.E. Project Viva — A.P. Shah Institute of Technology, 2025–2026.*
