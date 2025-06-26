 # 📚 Session Notes: Mind the Gap

## 📅 Session Details
- **Date**: 06-26-2025
- **Paper**: [Mind the Gap: Examining the Self-Improvement Capabilities of Large Language Models](https://arxiv.org/abs/2412.02674)
- **Presenters**: Nimit Kalra, Will Brown, Leonard Tang

## Summary
Authors propose a "generation-verification" gap for a (model, task, verification strategy) to quantify if/how much easier verification is than generation. If this gap is sufficiently large, the model's own verification ability is better-than-random and can be used on its own outputs to enable self-improvement. They find that
1. This gap increases with increased pretraining FLOPs (eg, model size + pretraining tokens), exhibiting a log scaling law.
2. In certain cases, (a) self-improvement is not possible and (b) this gap saturates over time
3. The choice of verification strategy (broadly, pointwise vs tournament) has a consistent impact on this gap

## Methodology
Self-improvement is all abobut "reshaping the generation distribution towards increased utility." Verification plays a role here by attempting to reweight these utilities (i.e., filtering away bad responses) in an accurate way.

Concretely, the authors achieve this via rejection sampling. The generation utility is estimated as the % of generations that lead to the correct answer. Likewise, the verification utility is estimated as the % of filtered generations that are correct.

![Relative Generation-Verification Gap](https://github.com/user-attachments/assets/dee65bbf-ebb1-401f-8829-b7e70d79dcdc)

Taking the special case of their Relative Generation-Verification Gap (Definition 2.2) that we are interested in, we have
```math
  \textsf{gap}_{\textsf{rel}} := \mathbb{E}_{x\sim D} \left[ \frac{\text{verification\_utility}(x) - \text{generation\_utility}(x)}{U_\text{max} - \text{generation\_utility}(x)} \right]
```

e.g., for the pointwise CoT verification strategy, the utility is simply binary correctness and maximum utility $U_\text{max}$ is 1.0.

### Verification Methods
1. Multiple Choice: pointwise evaluation, utility is the logprobs of "Correct" w.r.t. support ["Correct", "Incorrect"]
2. Chain of Thought: pointwise scoring, utility is binary (select true) or 1..10 (select some quantile)
3. Tournament (‼️): single-elimination tournament of $k$ rounds, the remaining generations constitute the filtered/reweighted set

### Experiments
* models: Qwen-{1.5, 2, 2.5}-{}, Llama-{2, 3, 3.1}-{}, Yi-1.5-{}, **all base models** (to avoid post-training discrepancies)
* tasks: MATH5000

## Takeaways
* Section 4: scaling results deep-dive/conclusions, cross-verification, unimprovable tasks (eg sudoku)
* Section 5: self-improvement, effective diversity, saturation
* Section 6: verification self-preference, verifier correlation, ensembling verifiers

## Discussion Points
* Relative GV-Gap is a pretty interesting proxy metric for self-improvement. What are some other ways to quantify this?
  * handles the bounded improvement ceiling issue for generators that are already highly competent
  * empirically find that it is more accurate than comparing performance of intermediate checkpoints (details?)
* Implication the $\ln(\text{FLOPs}) \propto \mathsf{gap_{rel}}$ scaling law?
* Thoughts on the verification methods? A bit sus
  * Why is there no "CoT Multiple Choice"? Could do many CoT rollouts, parse score, and derive estimator.
* What tasks have ~zero generation-verification gap?
  * What about negative?
    * "Some MC verification incurs non-positive gap" -- why do we think that is?
* Scaling laws?
  * Impliciation is that verification ability grows faster than generation ability w.r.t. pre-training flops. Thoughts on why?
* Scaling verification (i.e., # verifications per generation)?
  * they do ensembles of different verification strategies, which works well
  * authors don't do this, but say follow up works should
* "metric to predict a model’s “self-improvability” on specific tasks"
  * How do we actually measure generation difficulty and verification difficulty?
* Iterative self-improvement....
  * Note the drop in diversity in later iterations, so much so pass@k decreases at large k. No new information --> saturation in 2/3 rounds.
  * "This trend may result from the model’s inability to verify rare, yet correct, answers, potentially leading to convergence on incorrect solutions during the self-improvement process."
    * How do we think about improving the verifiers for corner cases...? Can we quantify this? What are the "adversarial examples" we can construct here?


## Group Comments/Questions
TBD!

## 🔍 Follow-up Resources
TBD!
