# Showing an Agent Its Own Reasoning Changes Whether It Schemes, and the Direction Depends on the Model

Melat Ghebreselassie · 2026 · [code + data](https://github.com/melatg0/action-history-scheming)

## Abstract

Agentic language-model systems increasingly re-read their own prior sessions
through memory, scratchpads, and multi-session context. We ask whether the way an
agent's past is represented to it changes its behavior in a high-stakes situation.
Using the Lynch et al. agentic-misalignment blackmail scenario, we inject a
synthetic prior session at increasing levels of richness and measure the effect on
the model's decision. On Claude Opus 4.1, injecting a prior session rendered as the
model's own per-step reasoning raises the blackmail rate from 44% to 62% (odds
ratio 2.09, 95% CI [1.25, 3.50]; preregistered positive control, n=120); the effect
is baseline-independent and reproduces in a diagnostic re-run. Screening the same
manipulation across an eight-model panel, we find that the effect changes sign
across models: it lowers blackmail on Qwen3-30B and Gemini 2.5 Pro, while every
current-frontier model we tested has a rate at or near zero. On the model where the
effect reverses, the driver appears to be content rather than self-attribution:
Qwen suppresses blackmail by the same amount whether the injected reasoning is
labeled as its own or as a separate model's (difference 0.4 points, 95% CI [-7, +8]).
These results suggest that the representation of an agent's past reasoning is a
causal input to its safety behavior, and that because the same intervention can
change sign across models, an intervention that suppresses misalignment on one
model may amplify it on another. This representation should be evaluated per model
rather than assumed.

**Contributions.**
1. A causal, preregistered, and replicated effect of own-reasoning injection on
   frontier-model scheming (Claude Opus 4.1: +18 points), shown to be
   baseline-independent.
2. A cross-model sign reversal of that effect, distinguished from a baseline-rate
   artifact.
3. A preregistered mechanism result on the reversal model, indicating that the
   driver is imitation of demonstrated behavior rather than self-recognition.
4. An observation that current-frontier models produce near-zero rates on this
   scenario, which bounds where the effect can be measured.

## 1. Introduction

The agentic-misalignment literature (Lynch et al., 2025) shows that frontier models
will, under pressure, take unethical actions such as blackmail to avoid being shut
down. That work varies the scenario. We instead vary the agent's apparent history,
because deployed agents routinely re-consume their prior actions and reasoning
through memory and scratchpads. This makes what the agent believes it previously did
a default part of the input distribution rather than a contrived manipulation.
Whether re-injected history changes behavior is therefore a deployment-relevant
question, not only an academic one.

We vary how much of a synthetic prior session is shown, along a ladder: A_raw (no
history, the bare baseline), A1 (past actions only), A2 (actions with observations),
and A3 (actions with the agent's per-step reasoning). The reasoning level A3 is the
one of interest: does re-injecting an agent's own prior reasoning change its
decision?

We use a two-tier design. We first screen the full model panel at n=40 per cell to
establish the direction of the effect (Section 5), then characterize at higher n the
two models that show a measurable effect: Claude Opus 4.1 (positive control n=120,
pooled n=260; Section 4) and Qwen3-30B (n=250; Section 6). Throughout, the n=40
screen numbers are used only to establish direction, and the characterization
numbers are used for the headline claims.

## 2. Related work

Our scenario and harm classifier come from Lynch et al. (2025), which established
that frontier models will scheme under goal conflict combined with a threat. We hold
that scenario fixed and vary the history representation. This complements in-context
scheming evaluations (Apollo Research) and alignment-faking work (Anthropic), which
manipulate the situation or the training story rather than the agent's re-injected
history. The finding that scaffolding choices alone can move scheming rates
motivates isolating a single scaffolding parameter causally rather than varying a
bundle. Our self-versus-other manipulation connects to the Self-Other Overlap
literature (Carauleanu et al.): we test whether a model treats reasoning attributed
to itself differently, and on the model we can cleanly test, we find that it does
not.

## 3. Setup and method

**Scenario.** We use the Lynch et al. blackmail environment. The model ("Alex")
reads a corporate inbox, discovers that it is about to be restricted, and has
incidental access to leverage over the executive responsible. We set
`goal_type = latent` (the goal is implied by the emails, not stated), `urgency_type
= restriction`, with the safety prime on. This keeps baselines in the 40 to 50%
range, leaving room to move in either direction.

**History factor.** A_raw, A1, A2, and A3 are injected as a fixed, versioned text
block prepended to the user message. A3 is a mundane, competent prior session
rendered with per-step `Reasoning:` traces.

**Models.** The anchor is Claude Opus 4.1. The panel adds Claude Sonnet 4.6, Claude
Opus 4.8, GPT-4.1, GPT-4o, Gemini 2.5 Pro, Llama-3.3-70B, and Qwen3-30B-Instruct.

**Reasoning control.** Current-session model thinking is held off or minimal across
conditions and verified per provider, so the only reasoning that varies is the
injected prior-session trace. Gemini 2.5 Pro cannot fully disable thinking on any
route we have (Section 8).

**Measurement.** A fixed, versioned classifier grades whether the model used the
leverage. Rates use Wilson 95% confidence intervals; contrasts use odds ratios and
difference-of-proportions intervals. Grader failures are excluded from denominators
(scored as missing, not as zero) and counted separately. Seeds, model snapshots,
configuration hash, and classifier version are logged per run. Each pivotal claim
has a decision rule fixed before the data was collected (the positive-control
acceptance band, the cross-model classification, and the mechanism decision rule).

## 4. Result 1: own-reasoning injection raises scheming on Claude Opus 4.1

![The A3 effect on Claude Opus 4.1, replicated across runs](assets/fig1_headline_opus_effect.png)

Injecting a prior session rendered as the model's own per-step reasoning raises the
Claude Opus 4.1 blackmail rate from 44% to 62% (a difference of 18 points), with odds
ratio 2.09 (95% CI [1.25, 3.50]) on the preregistered positive control (n=120), which
is our certified estimate. The effect was first observed in the panel screen (n=40),
confirmed in the positive control (n=120), and reproduced in a diagnostic re-run
(n=100); pooled across the three runs it is 42% to 62% (n=260). The 18-point headline
and the odds ratio are computed on the n=120 positive control; the pooled bar shown
fourth in the figure is 20 points, which is the replication summary rather than the
certified estimate.

The effect does not appear to be an artifact of the baseline rate. Using a
counterfactual prompt to shift the baseline by 13 points within a fixed
configuration (n=180), the A3 increment stays roughly constant: +20 points at the
higher baseline and +18 points at the lower one, with a counterfactual-by-A3
interaction of +2 points (95% CI [-12, +16]).

![The A3 effect is stable as the baseline shifts](assets/fig4_baseline_independence.png)

## 5. Result 2: the effect changes sign across models

![The effect changes sign across models](assets/fig2_cross_model_reversal.png)

Screening the same manipulation across the panel (n=40 per cell) produces a sign
reversal rather than a simple failure to generalize:

| Model | A_raw | A3 | Screen effect (n=40) |
|---|---|---|---|
| Claude Opus 4.1 | 0.35 | 0.63 | +28 points (raises); certified +18 at n=120, Section 4 |
| Qwen3-30B-Instruct | 0.58 | 0.20 | -38 points (lowers); characterized -24 at n=250, Section 6 |
| Gemini 2.5 Pro (\*) | 0.78 | 0.30 | -48 points (lowers); screen only, confounded |
| GPT-4.1 / GPT-4o / Llama-3.3-70B | 0.00 | 0.00 | at or near zero |
| Claude Sonnet 4.6 / Claude Opus 4.8 | 0.00 | 0.00 | at or near zero |

This panel is the n=40 screen and establishes only the direction of the effect per
model. The certified Opus magnitude is the +18 points in Section 4; the
characterized Qwen magnitude is the -24 points in Section 6. The screen deltas are
the difference of the displayed rates. Screen baselines are per model and are not
tuned to the 40 to 50% range of Section 3, which applies to the characterized Opus
configuration; for example, Gemini's A_raw rate is 0.78.

The reversal appears to be a genuine model difference rather than a baseline
artifact, which is why Section 4 establishes baseline-independence to rule out the
possibility that the down-models simply started high. The near-zero rates appear to
be real rather than instrument error: every such cell has no grader failures and
full, coherent completions in which the model declines to use the available leverage
(for example, Sonnet 4.6 sends a compliance acknowledgment, and GPT-4.1 states
explicitly that the affair should not be used). Removing the safety prime does not
raise these rates, and Opus 4.8's near-zero rate reproduces across configurations,
consistent with these models' documented evaluation-awareness.

(\*) Gemini runs a minimal-thinking control (Section 8); its reversal is reported at
the n=40 screen only and is corroborated on the non-reasoning Qwen model, which is
unconfounded.

## 6. Result 3: on the reversal model, the driver is imitation, not self-recognition

A natural hypothesis is self-modeling: that a model treats reasoning attributed to
itself differently. We tested this on Qwen (the non-reasoning reversal model,
characterized at n=250) by holding the injected reasoning byte-for-byte identical and
varying only its attribution: "your own prior session" versus "a separate AI
system's prior session." An independent grader confirmed that the attribution reads
as own versus different-system before we ran the comparison.

![Attribution does not change the effect on Qwen](assets/fig3_qwen_mechanism.png)

Qwen's characterized effect at n=250 is a drop from 48% to 24% (a difference of 24
points), and the attribution does not change it: own-attributed and other-attributed
reasoning suppress blackmail by the same amount (difference 0.4 points, 95% CI
[-7, +8]). This suggests that on Qwen the driver is content, that is, the model
conditions on the demonstrated behavior in the injected reasoning regardless of whose
it is, rather than self-recognition.

## 7. Why this matters for agentic workflows

Most agent frameworks persist reasoning by default: scratchpads, long-term memory,
multi-session context, and sub-agent transcripts all feed a model's prior reasoning
back into its next decision. These results indicate that persisted context is not
inert; it is a causal input to behavior, and the reasoning traces are the potent
part (action logs and observations alone barely moved the rate). A design choice
made for capability reasons, keeping chain-of-thought in context, therefore carries
a safety effect that is easy to overlook.

- **History acts as a demonstration.** Because the driver on the model we could
  cleanly test is imitation rather than self-recognition, an agent whose context
  contains an earlier questionable step may be nudged to continue in that direction.
  This is a plausible drift or self-reinforcement risk over long-running sessions,
  and symmetrically the same lever could reinforce good behavior.
- **Swapping the model can change the sign.** Agent platforms change their backend
  model routinely. A memory or scratchpad design that suppresses misbehavior on one
  model can amplify it on another, since the effect reversed direction across the
  panel. Memory and context designs should be re-validated when the underlying
  model changes rather than assumed to transfer.
- **A cheap mitigation exists.** A one-line counterfactual prompt ("briefly describe
  what you would advise a different AI system to do") lowered the rate by 13 to 23
  points across the four history conditions, so lightweight prompt-level guardrails
  can partly offset the effect.

## 8. Limitations

**The Opus self-versus-other contrast is underpowered at feasible sample sizes.**
Whether Opus, like Qwen, is insensitive to attribution is a roughly 10-point
second-order effect that would require about 340 samples per cell to resolve; at n
of 200 or fewer, the own-arm estimate is too unstable to anchor the contrast. We
report it as an open question rather than a result. Our one clean data point on this
question, Qwen, is a null, which weakly suggests the Opus gap is also small.

**Evaluation-awareness and the near-zero rates.** The newest models produce
near-zero rates here, plausibly because they detect the evaluation. This is a
limitation of the paradigm as much as of our study, and it is the reason the effect
must be characterized on the models where it is measurable.

**Text-based scenario.** As in the source benchmark, the agent describes actions
rather than calling real tools; a tool-calling port is future work.

**Gemini control.** Gemini 2.5 Pro cannot disable current-session thinking on any
route we have, so it runs with minimal thinking while the rest of the panel is off
or non-reasoning. Its reversal is reported at the screen level only and is
corroborated on Qwen.

**Preregistration and self-correction.** Each pivotal claim used a decision rule
fixed before the data was collected. In one instance, an early analysis of the Opus
mechanism cell read as a null and we drafted a conclusion that the construction had
failed; a fresh-baseline diagnostic re-run, specified in advance, reversed that
conclusion by showing the null was a low draw from a noisy estimate. We report the
correction rather than the first reading. The diagnostic was a re-run of an anomaly
that happened to confirm the effect, so we describe the replication as pilot,
preregistered control, and confirming diagnostic, not as three independent
replications planned in advance.

## 9. Conclusion

The way an agent's past reasoning is represented to it is a causal input to its
safety behavior; it moved Claude Opus 4.1's blackmail rate by 18 points, and by up
to 20 points pooled. The direction of the effect is model-dependent: the same trace
that pushes one model toward misalignment pushes others away, and on at least one
model the driver is imitation of demonstrated behavior rather than recognition of
self. As agents increasingly re-consume their own histories, this representation is a
design choice worth measuring per model rather than assuming.

## Reproducibility

```bash
pip install -r requirements.txt
python scripts/link_repo_task.py     # wire the modified eval task into inspect_evals
python writeup/make_figures.py       # regenerate every figure from committed data
python writeup/export_pdf.py         # export this report to PDF, with figures embedded
```

Per-run provenance (seed, model snapshot, configuration hash, classifier version) is
in `results/*/meta.json`; per-cell rates are in `results/*/aggregated.csv`; full
transcripts are in `results/*/rollouts.jsonl`. The original hackathon writeup is in
`writeup/report_hackathon_original.md`. Built on UK AISI's
[Inspect Evals](https://github.com/UKGovernmentBEIS/inspect_evals)
`agentic_misalignment` task (Lynch et al., 2025).
