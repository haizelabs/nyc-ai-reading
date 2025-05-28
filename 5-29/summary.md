 # 📚 Session Notes: SFT Memorizes, RL Generalizes

## 📅 Session Details
- **Date**: 05-29-2025
- **Paper**: [SFT Memorizes, RL Generalizes: A Comparative Study of Foundation Model Post-Training](https://arxiv.org/abs/2501.17161)
- **Presenters**: Nimit Kalra, Will Brown, Leonard Tang
- **Scribe**: Nimit Kalra

## Summary
In this work, the authors set-up controlled experiments to investigate the impact of RL vs SFT on generalization across two bespoke tasks. Contrary to the common nomenclature in a supervised learning setting, the in/out-distribution are not from the same underlying data distribution, but are rather visual/rule-based extensions within the same environment. Under this setting, the authors find that RL generalizes to new task rules and visual artifacts, whereas SFT degrades severely.

The authors assert that this is not some quirk of the data that is seen during RL rollouts, but rather a fundamental property of the weight updates that SFT performs.

## Discussion Points
* Generalization in the RL literature has a different connotation than in the standard supervised learning setting. How is this more or less applicable in the RL environments that people care about today?
* When/is SFT a necessary first step in RL? Qwen models have something in the pre-training distribution that makes them exceptional at instruction-following for reasoning; this has made them a popular initialization base model for RL.

### Tangential Discussion Points — *What does RL Actually Learn?*
* https://arxiv.org/abs/2505.11711
* https://www.interconnects.ai/p/reinforcement-learning-with-random

## Methodology
For the SFT experiments, they simply finetune on prompt-response dialogues. Hence, there are no verifier/revision steps as in the RL setup. They do ablate with these in Appendix C.1., but find little generlization improvement.

For all RL experiments, they first SFT and then apply PPO on outcome-based rewards. They use the Sequential Revision Formulation [(Snell et al., 2024)](https://arxiv.org/pdf/2408.03314) to construct each input at step $t$; this means $v^\text{in}_t$ includes all prior policy attempts and associated verifier outputs. It seems like they have a fixed number of maximum verification steps per trajectory, regardless of the final attempt is a success.

## Tasks
Authors focus on two bespoke tasks — arithmetic reasoning (`GeneralPoints`) and open-world visual navigation (`V-IRL`) and produce a visual-reasoning modification (denoted `-VL`). Furthermore, they use both rule and visual variations to construct a held-out out-of-distribution test set to assess generalization performance.

## Experiments / Takeaways
* Base model: `Llama-3.2-Vision-11B`

### Takeaways
* RL generalizes to unseen rules, whereas SFT simply memorizes the rules seen.
* SFT important for RL training, as a first step to encourage instruction-following for the task's reasoning format.
> Note that due to the difference in backbone model, our results do not contradict with DeepSeekAI et al. (2025), which suggests that SFT is unnecessary for downstream RL training.

* Even training SFT on sub-optimal trajectories (i.e., "wind-y" attempt + verifier multi-turn dialogues) doesn't solve the generalization problem. See Appendix C.1.
* Scaling up verification improves generalization.

## Group Comments/Questions
TBD!

## 🔍 Follow-up Resources
TBD!
