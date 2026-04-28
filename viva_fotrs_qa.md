# Viva Q&A — Factored Optimal Transport Reward Shaping (FOTRS)
*Focused evaluator questions on the FOTRS module with clear, simple answers.*

---

## Q1. What is FOTRS in one simple sentence?

**Answer:** FOTRS is a multi-agent reward system that gives each specialist agent a personalized, mathematically fair score based on how much its actual behavior deviates from ideal behavior—measured using optimal transport distances over a coordination graph.

**Analogy:** In a group project, instead of giving everyone the same grade, FOTRS grades each student by comparing their actual work to what ideal work looks like, and it only compares students who actually worked on the same section.

---

## Q2. What problem does FOTRS solve? Why was it needed?

**Answer:** It solves the **cooperative credit assignment bottleneck** in Multi-Agent Reinforcement Learning (MARL).

- In a portfolio, many sector agents (Banking, IT, Pharma, etc.) work together. The only signal they usually get is the **total portfolio return**.
- If the portfolio loses money, *every* agent gets punished equally—even if only the Energy agent made a bad call. This is unfair and confusing.
- Agents cannot learn properly because they do not know which of their own actions caused success or failure. This leads to free-riding, suboptimal policies, and slow convergence.

FOTRS fixes this by giving each agent a **localized, dense reward signal** that isolates its own contribution.

---

## Q3. What is Optimal Transport, and why did scientists invent it?

**Answer:** Optimal Transport (OT) is a mathematical theory about the cheapest way to move piles of dirt from one place to another.

- **History:** 
  - **Gaspard Monge** (1781, French mathematician) first posed the problem: how to optimally rearrange soil when building fortifications with minimal work.
  - **Leonid Kantorovich** (1942, Soviet mathematician/economist) relaxed the problem into a linear program. He won the **Nobel Prize in Economics in 1975** partly for this work.
  - **Marco Cuturi** (2013, NeurIPS) made it practical for machine learning by adding **entropy regularization**, creating the **Sinkhorn algorithm** that solves OT in near-linear time instead of exponential time.

- **Why it matters for us:** In finance, returns are not simple numbers—they are full probability distributions. Wasserstein distance (the metric from OT) respects the shape of these distributions, handles heavy tails and non-Gaussian data far better than correlation or Euclidean distance.

**Analogy:** If you have two histograms of daily returns, Euclidean distance only compares their averages. Wasserstein distance asks: *"How much effort does it take to reshape one histogram into the other?"* It captures the entire distribution, not just one point.

---

## Q4. How is FOTRS different from standard MARL methods like QMIX or MAPPO?

**Answer:**

| Feature | QMIX / MAPPO | FOTRS (Ours) |
|---------|-------------|--------------|
| **Reward signal** | Shared global scalar (everyone gets the same portfolio return) | Personalized reward = Global return − Local Wasserstein penalty |
| **Credit assignment** | Implicit via value decomposition (QMIX) or centralized critic (MAPPO) | Explicit via pairwise distributional distances on a coordination graph |
| **Convergence proof** | Empirical best-effort | Guaranteed Nash equilibrium on tree-structured graphs (Theorem 3) |
| **Sample complexity** | Exponential in total joint dimension | Polynomial in local edge dimension |
| **Sharpe Ratio** | 0.64 (QMIX), 0.73 (MAPPO) | **0.89** |
| **Convergence speed** | 185–210 episodes | **78 episodes** (2.8× faster) |

In plain terms: QMIX and MAPPO try to guess who deserves credit by looking at the total outcome. FOTRS *mathematically measures* each agent's deviation from ideal coordination and penalizes exactly that.

---

## Q5. What is the coordination graph, and why did you use Graphical Lasso?

**Answer:**

- **Coordination graph** G = (V, E) is a network where:
  - Each **node** is a sector agent (IT, Banking, Pharma, Energy, FMCG, Metals, Realty, Auto).
  - Each **edge** connects two sectors that actually influence each other (e.g., Energy and Metals are linked because metal prices depend on energy costs).

- **Graphical Lasso** is an algorithm that estimates a **sparse precision matrix** (inverse covariance matrix) by applying L1 regularization. It automatically sets weak correlations to exactly zero, keeping only the strongest conditional dependencies.
  - Without this sparsity, the graph would be a dense complete graph K8 with 28 edges, and computation explodes.
  - With Graphical Lasso, we get ~12–17 edges, making the problem tractable.

**Analogy:** Instead of forcing every student to talk to every other student in a group project, Graphical Lasso figures out who actually needs to coordinate (e.g., the writer needs to talk to the editor, but not the cover designer).

**Critical result:** Our ablation showed that removing the sparse graph and using the dense complete graph (K8) **crashed the Sharpe ratio from 0.89 to 0.71**, proving the sparse topology is mathematically necessary.

---

## Q6. What is the Factored Wasserstein Distance, and why factorize it?

**Answer:**

The standard Wasserstein distance compares two full joint probability distributions over all agents and all states. For 8 agents, this is an exponentially huge computation.

**Factored Wasserstein Distance** breaks this monster problem into small, local pieces:

```
W_fact(ρ^π, ρ*; G) = Σ_(i,j)∈E  w_ij · W_cij(ρ^π_ij, ρ*_ij)
```

Instead of one giant transport problem, we compute |E| small transport problems—one per graph edge. Each small problem only looks at the marginal distribution of two neighboring agents.

**Why this is powerful:**
- **Computational:** Solving 12 small problems is infinitely easier than solving one 8-dimensional joint problem.
- **Statistical:** Sample complexity drops from **exponential** in total agents to **polynomial** in local neighborhood size. This is an exponential reduction.
- **Decentralized:** Each agent only needs to know about its neighbors, not the entire system.

**Analogy:** Instead of planning traffic for an entire country at once, you plan traffic district by district, but only connect districts that actually share a road.

---

## Q7. What is the reward shaping formula, and how does it give fair credit?

**Answer:** Each agent i receives:

```
r_i^FOTRS(t) = R_global(t) − λ Σ_j∈N(i) w_ij · W_cij(ρ̂^π_ij, ρ*_ij)
```

Breaking this down:
- **R_global(t):** The total portfolio return. Everyone gets this base score.
- **λ:** A tuning knob controlling how strongly we penalize misalignment.
- **N(i):** Agent i's neighbors in the coordination graph.
- **w_ij:** Edge weight = strength of dependency between sectors i and j.
- **W_cij(...):** The Wasserstein distance between agent i's actual pairwise behavior and the ideal target behavior on edge (i,j).

**How it gives fair credit:**
- If the Banking agent coordinates perfectly with its linked sectors, its Wasserstein penalty is near zero. It keeps almost all the global reward.
- If the Energy agent misaligns with Metals, *only* the Energy-Metal edge penalty increases. Energy gets a lower shaped reward, while unrelated agents like Pharma are unaffected.

**Analogy:** Imagine a relay race. Traditional MARL gives the entire team the same time. FOTRS measures each runner's deviation from perfect form and subtracts only their own mistake penalty from the team score.

---

## Q8. What is the Sinkhorn algorithm, and why do you need entropy regularization?

**Answer:**

- **The problem:** Computing exact Wasserstein distance requires solving a massive linear program (the Kantorovich formulation). For large-scale ML, this is too slow.
- **Cuturi's solution (2013):** Add a tiny bit of **entropy regularization** to the transport cost. This smears the optimal transport plan slightly, making the problem strongly convex.
- **Sinkhorn algorithm:** An iterative matrix-scaling algorithm (similar to adjusting rows and columns of a table until they sum to 1) that converges in near-linear time.
  - Complexity drops from exponential/cubic to roughly **O(L²/ε)** where L is the support size and ε is the regularization strength.
  - It is fully differentiable, so gradients flow smoothly through neural networks.

**Analogy:** Exact OT is like finding the single perfect path through a maze by checking every possibility. Entropy-regularized OT is like adding a little "fuzziness" so the algorithm quickly finds an extremely good path using a simple water-filling approach.

---

## Q9. You mention Nash Equilibrium convergence. What does that mean practically?

**Answer:**

- In game theory, a **Nash Equilibrium** is a state where no agent can improve its reward by changing its strategy alone, assuming others keep their strategies fixed.
- In standard MARL, agents often oscillate forever because they keep reacting to each other's changing policies. This is called **policy chasing** or non-convergence.

**Our guarantee (Theorem 3):** When the coordination graph is a tree, FOTRS defines an **exact potential game**. This means there exists a single global "scoreboard" (the negative factored Wasserstein distance) that every agent implicitly agrees to maximize. Any sequence of best-response updates must climb this scoreboard and eventually stop at a Nash Equilibrium.

**Practical meaning:** The 8 sector agents will **not** fight each other indefinitely. They will settle into a stable, mutually optimal portfolio allocation. In our experiments, this convergence happened in **78 episodes**, compared to 185 for MAPPO.

**Analogy:** A potential game is like hikers climbing a single mountain. They may take different paths, but they all know that going upward is good, and they will eventually reach the same peak. Non-potential games are like hikers on separate mountains who keep jumping to each other's peaks forever.

---

## Q10. What happens if the graph is not a perfect tree? Real markets have cycles.

**Answer:** We address this in **Theorem 4** (Bounded General Graph Approximation).

- Real coordination graphs often have cycles (e.g., Banking ↔ Realty ↔ Construction ↔ Banking).
- We extract the **maximum-weight spanning tree** from the full graph. This keeps the strongest dependencies and drops the weakest ones.
- The theorem proves:
  ```
  W_tree ≤ W_full ≤ W_tree + (small error bound)
  ```
- In practice, for 8 Indian sectors, the top 7 edges by precision weight capture **over 85%** of total conditional dependency. The approximation gap is tiny.
- Our ablation confirmed this: using the full dense graph (28 edges) actually **hurt** performance because it destroyed the sample complexity gains. The sparse tree/graph structure is essential.

---

## Q11. Why not just use "Difference Rewards" (counterfactual baselines)? That also solves credit assignment.

**Answer:** Difference rewards are theoretically elegant but practically broken for our setting:

- **Difference reward:** r_i^diff = R_global − R_global^(-i)  (global reward minus global reward *without* agent i).
- **The problem:** Computing R_global^(-i) requires simulating the entire system with agent i removed or replaced. In a continuous-action portfolio space, this means running an extra full episode for every agent, at every step. That is **computationally prohibitive**.
- **FOTRS advantage:** The Wasserstein penalty acts as a **geometric difference reward** without counterfactual simulation. It measures distributional deviation directly from occupancy statistics already collected during normal training. No extra rollouts needed.

**Analogy:** Difference rewards ask "what would the team score be if I sat out this game?" FOTRS asks "how far is my running pattern from the ideal pattern?" The second question can be answered by watching tape, without replaying the match.

---

## Q12. What is the policy gradient formula in FOTRS, and why is it decentralized?

**Answer:** The gradient for agent i is:

```
∇_θi u_i = − Σ_j∈N(i) w_ij · E[ f*_ij(s_ij, a_i, a_j) · ∇_θi log π_θi(a_i | s) ]
```

- **f*_ij:** The optimal Kantorovich dual variable (potential) from the Sinkhorn solver. It is computed once per edge and shared.
- **∇_θi log π_θi(a_i | s):** The agent's own policy score function (standard REINFORCE trick).
- **Decentralized:** Agent i only needs information from its immediate neighbors j ∈ N(i). It does not need to know what the Pharma agent is doing if Pharma is not connected to it in the graph.

**Practical benefit:** This allows institutional deployment where each trading desk (agent) only shares limited pairwise statistics with desks it actually trades with, preserving information privacy.

---

## Q13. Show us the empirical proof that FOTRS actually assigns credit correctly.

**Answer:** We validated this with a **Spearman rank correlation test**.

- We computed the pairwise Wasserstein distances (d_ij) for each edge in the coordination graph.
- We compared these distances to the **realized intra-sector crisis losses**.
- Result: **Spearman ρ_s = 0.51** (p < 0.01).

**What this means:** When the Energy-Metal edge showed a large Wasserstein distance (meaning those two agents were poorly coordinated), the actual realized losses in those sectors during crisis regimes were significantly higher. The model successfully detected and penalized toxic misalignment *before* the global portfolio crashed.

In short: the local Wasserstein penalties genuinely predict where the portfolio will bleed, validating that credit is being assigned to the right agent pairs.

---

## Q14. How does FOTRS perform in a market crash?

**Answer:** This is where FOTRS shines most.

- During crisis regimes, traditional methods like Risk Parity suffered a **67.3% maximum drawdown** because they assume stable correlations. In crashes, correlations spike to +1, and diversification evaporates.
- FOTRS maintained drawdown at only **34.0%** because:
  1. The coordination graph captures conditional dependencies. When correlations spike, the edge weights adapt, and agents collectively shift to defensive postures rather than independently panic-selling.
  2. The conformal prediction layer widens uncertainty sets during volatile regimes, signaling the system to reduce aggressive exposures.
  3. The Mahalanobis transport cost penalizes high-volatility reallocations, preventing agents from making rash moves in turbulent markets.

---

## Q15. Can you summarize the entire FOTRS contribution for a non-technical evaluator?

**Answer:**

> "When multiple AI agents manage a portfolio together, the hardest problem is figuring out who to blame when things go wrong. Existing methods give everyone the same score, so nobody learns. 
>
> We invented FOTRS, which compares each agent's behavior to an ideal target using a 200-year-old mathematical idea called Optimal Transport. It builds a network map of which sectors actually affect each other, then gives each agent a personalized grade based only on its own connections. 
>
> Mathematically, we proved the agents will settle into a stable team strategy instead of fighting forever. In experiments on Indian stock data, our 8-agent team achieved a Sharpe ratio of 0.89—22% better than the best existing method—and learned 2.8× faster, while cutting crash losses in half."

---

## Quick Reference: Key Numbers for FOTRS

| Metric | Value |
|--------|-------|
| Sharpe Ratio | **0.89** |
| vs. MAPPO baseline | **+22%** |
| vs. QMIX baseline | **+39%** |
| Max Drawdown | **34.0%** |
| Training Episodes to Converge | **78** (MAPPO: 185) |
| Convergence Speedup | **2.8×** |
| Spearman ρ_s (credit validation) | **0.51** (p < 0.01) |
| Dense Graph Ablation Sharpe | **0.71** (vs. 0.89 sparse) |

---

*Prepared for B.E. Project Viva — A.P. Shah Institute of Technology, 2025–2026.*
