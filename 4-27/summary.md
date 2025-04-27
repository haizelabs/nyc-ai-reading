 # 📚 Session Notes: Inference-Time Scaling for Generalist Reward Models

## 📅 Session Details
- **Date**: 04-27-2025
- **Paper**: [Inference-Time Scaling for Generalist Reward Models](https://arxiv.org/abs/2504.15046)
- **Presenters**: Nimit Kalra, Will Brown, Leonard Tang
- **Scribe**: Leonard Tang

Inference-Time Scaling for Generalist Reward Models introduces Self-Principled Critique Tuning (SPCT), a new method for training generalist reward models. The key claim is these GRMs perform well on non-verifiable domains -- key for extending reasoning models beyond the common domains of math and programming -- and are furthermore amenable to inference-time scaling.

## 💡 Discussion Highlights
### Generalist Reward Modeling via Principles (Rubric) Generation
DeepSeek-GRM factors the reward modeling problem into two steps:
1) Generate a set of query-specific principles (rubric) that can be used to evaluate a given sample
2) Based on these principles, generate rationales ("critiques") to produce a final pointwise numeric rating (reward)

The hope is that the generated principles induce more diverse rationale generation, and ultimately higher quality rewards.

### Rejective Fine-Tuning
RFT is a STaR-like SFT learning algorithm with two rejection criteria:
1) Reject generated samples that predict the incorrect rating
2) Reject questions that are too easy; where all generated samples predict the ground truth rating

In the style of STaR rationalization: in 1), RFT optionally injects the ground truth rating into a new query to re-generate an explanation & rating.

At each iteration of RFT, the collected successes and failures -- both absent of rejected trajectories -- form an offline dataset that is used to SFT `Gemma-2-27B`.

#### Group Comments
- RFT is very reminiscent of STaR, but does not waste training compute on "easy" inputs that the base model already knows how to score.

### Rule-Based Reinforcement Learning
RFT solves the cold-start problem, but the RFT'd GRM needs help generalizing. To assuage this, the authors use straight GRPO with the following reward function:

$$
\hat{r_i} = \begin{cases}
1, & \text{if } n \geq 2 \text{ and } \forall i' \neq j', \quad S_{j'} > S_{i'}, \quad j' = \arg\max_l\{r_l\}_{l=1}^n \\
1, & \text{if } n = 1 \text{ and } S_1 = r_1 \\
-1, & \text{otherwise}
\end{cases}
$$

where $n$ is the number of responses to score, $o_i$ is the $i$-th sampled output from GRM given the query $x$ (input and response to score), $S_i$ is the extracted reward (e.g. via regex) from $o_i$, and $r_i$ is the ground truth reward. The selected remaining $k_{meta}$ responses are then used for voting.


#### Group Comments
- Q: Why is RFT alone insufficient?
    - A: Hypothesis is that RFT memorizes to some degree, and RL is required for generalization. See [here](https://arxiv.org/abs/2501.17161) for more.
- Q: Why is there no reward for formatting the response?
    - A: Format rewards are extremely helpful for small & dumb models, but larger models can readily follow formatting instructions. 
    - A: The high KL coefficient is sufficient to preserve appropriate formatting.
- It's interesting that the reward function in the $n \geq 2$ case only requires $ \arg\max_l\{\hat{r}_l\}_{l=1}^n = \arg\max_l\{r_l\}_{l=1}^n $, rather than requiring the predicted ordering to match the ground truth ordering.
    - Given that preference benchmarks in the paper and in general are built around pairwise comparisons, it's hard to ascertain if this is sufficient in the more general $n \geq 3$ setting.

### Inference-Time Scaling

At inference-time, the authors scale compute by parallelizing GRM rollouts and selecting the best sampled rationale + reward by:
1) Summing the rewards of each rollout ("Voting")
2) Applying a Meta-RM to select the best rationale + reward

A Meta RM is trained on a dataset comprising both successful and failed rollouts from RFT and rule-based RL using binary cross-entropy loss. This is applied at inference-time to prune low-quality principles and rationales produced by the GRM. 

#### Group Comments
- The Meta RM is trained on in-distribution data...which is suspicious. 
    - It is possible that the Meta RM need only be directionally correct given, and the voting stage may eliminate any errors.
- It's unclear whether the Meta RM ingests the query, response, rationale, and reward, or just the rationale and reward. 
    - If the latter, then the Meta RM is likely more useful given its rating ability is not pinned to any specific set of underlying data to be scored.

## 🔍 Follow-up Resources
- WIP: Haize OS implementation
- Brendan's early experiments on using GRM-style prompting (principles $\to$ rationales & reward) with LLM judges as a reward for training debate and joke generation models [here](https://github.com/brendanhogan/DeepSeekRL-Extended/tree/llm_rewards_grm)
