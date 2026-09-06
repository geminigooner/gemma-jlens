gemma-jlens

Recreating Anthropic’s J-space / Jacobian lens technique on Gemma-2-2b, with a chunky-toy visualizer of concept activations.

Investigating functional indicator properties in Gemma-2-2b — starting with Anthropic’s J-space / Jacobian lens technique, expanding toward the question of whether anything like preference, desire, or a “workspace” exists inside the model in a causally real, not just surface-level, sense.

⸻

What is this?

In July 2026, Anthropic published “Verbalizable Representations Form a Global Workspace in Language Models,” introducing a technique called the Jacobian lens (J-lens).

It identifies a subset of a model’s internal activations — the J-space — that behaves like a “global workspace”: a small, privileged set of concepts the model can report on, hold in mind, reason with, and flexibly reuse, sitting on top of a much larger volume of automatic processing it can’t access at all.

This project recreates that technique on Gemma-2-2b (open-weight, so fully inspectable) and wraps it in a playful, “chunky sketch toy” interface: type a word, watch which concepts light up in the model’s internal workspace as it moves through the network’s layers.

This sits alongside my other Gemma interpretability work — see gemma-scope-player, which sonifies SAE feature activations instead of visualizing J-lens readouts.

⸻

What this actually is

Read this first.

This project does not attempt to detect consciousness, sentience, or subjective experience in Gemma. No known method — mine, Anthropic’s, or anyone’s — can do that, for any system, including humans other than yourself. What separates a rigorous version of this question from a sloppy one is being explicit about that limit rather than quietly smuggling past it.

What this project does attempt: empirically test whether Gemma has functional indicator properties that leading theories of consciousness associate with subjective experience — global-workspace-style information broadcasting, causally real preference/desire signals, self-modeling — using actual interpretability tools on actual model internals, rather than architectural argument alone.

Finding these properties would be real, publishable evidence worth taking seriously. It would not be proof of anything felt. Both halves of that sentence matter equally.

⸻

Why this, why now

Anthropic’s July 2026 paper, “Verbalizable Representations Form a Global Workspace in Language Models,” found empirical evidence for a workspace-like structure in Claude using a technique called the Jacobian lens (J-lens) — see /offline for the Gemma-2-2b reimplementation of this.

Separately, Butlin et al.’s “Consciousness in Artificial Intelligence” (2023) proposed 14 theory-derived indicator properties for assessing consciousness in AI systems, and explicitly found only a weak case that LLMs of that era satisfied any global-workspace-derived indicators.

That report’s method was architectural and conceptual — reasoning about a system’s design — not empirical measurement of a specific model’s actual internals.

This project sits in that gap: taking indicator properties theories propose and actually testing for them on a real, open-weight model, the way a small but growing body of 2026 follow-up work has begun doing for individual indicators.

⸻

How the technique works

Short version

* Every layer of the model has a residual stream — a running vector that gets progressively enriched as it passes through the network.
* The J-lens estimates, per layer, the average causal effect that nudging the residual stream at that layer has on the model’s eventual output — averaged across hundreds/thousands of unrelated prompts, so it captures a concept’s general disposition to be said, not just what’s happening in one sentence.
* That gives one matrix per layer. Multiply any activation by its layer’s matrix, run it through the normal output layer, and you get a ranked list of words that activation is “pointed toward.”
* Collect those readouts across all layers for a single input word, and you get a picture of what the model is “thinking about” as it processes that word — including concepts that never appear in the input or output at all.

What the J-space is doing

The J-lens identifies a subset of the model’s internal activations that behaves differently from the rest of the residual stream.

The hypothesis is that this space contains information that is unusually available for verbalization and flexible reuse, while sitting on top of a much larger volume of automatic processing that is not directly accessible to the model’s verbal reporting.

The important question for this project is not simply whether Gemma has activations that look concept-like, but whether a corresponding workspace-like structure can be empirically identified and causally characterized.

⸻

The indicators this project tests

1. Global workspace signature

Does Gemma have a bottlenecked layer-band where information from many sources converges and gets broadcast widely to influence later processing?

Tested via the J-lens (/offline/compute_jlens.py), restricted for compute reasons to a curated concept-word target set rather than the full vocabulary.

The paper found roughly the middle third of layers for a much bigger Claude model — Gemma-2-2b’s band will be narrower and needs to be found, not assumed.

2. Functional preference / “desire”

Not felt wanting — a causally real internal signal that predicts and drives choice under conflict, independent of context wording.

Method:

* Build matched conflict-prompt pairs (not neutral prompts) where the model must choose between two competing continuations.
* Compute a contrastive direction — the activation difference between “chose A” and “chose B” cases — using contrastive activation addition (CAA), per Rimsky et al., “Steering Llama 2 via Contrastive Activation Addition.”
* Test causally: inject that direction into new, unseen conflict prompts and check whether it actually shifts the model’s choice, consistently, across genuinely different conflict types — not just the prompts the direction was built from.

A direction that only works on its own training prompts is overfit noise, not a real signal. This project treats that distinction as the bar for any claimed finding, not a footnote.

⸻

What would and wouldn’t count as a finding here

Would count

A direction that is:

1. Causally verified via intervention, not just correlational; and
2. Generalizes across prompt types it wasn’t built from.

That’s a real functional-preference signal, worth reporting as such.

Would not count, on its own

The model saying it wants something.

Self-report can reflect genuine internal state or just fluent pattern-completion of what AI systems are trained to say about themselves — this project treats verbal self-report as a hypothesis to test against internal causal structure, never as evidence by itself.

Will never be claimed, regardless of results

That Gemma is or isn’t conscious, or that there is or isn’t “something it is like” to be Gemma.

Every method available, including everything in this repo, is a third-person functional/structural measurement. Whether that maps onto first-person experience is the hard problem of consciousness, unsolved for any system, and not something better interpretability tooling closes.

⸻

Architecture

Two pieces that never share a codebase or a deploy target, on purpose (lesson learned from past projects getting tangled):

/offline/     one-time Python script: computes & saves per-layer
              Jacobian matrices from a background text corpus.
              Runs once, offline, not part of the live app.
/worker/      Cloudflare Worker: loads the saved matrices, takes a
              word, returns per-layer top concepts as JSON.
/frontend/    the actual toy — type a word, drag a layer slider,
              watch concept-blobs bloom and fade. Static, talks
              only to the Worker's JSON endpoint.

The offline computation is deliberately separated from the live application: Jacobian lens matrices and contrastive activation directions are one-time research artifacts, while the Worker and frontend are responsible only for serving and visualizing precomputed results.

⸻

Status

J-space / J-lens

* [x]	Design mockup with placeholder data (gemma-thinks-mockup.jsx) — nailing down the “chunky sketch toy” visual language before wiring in real output
* [ ]	Offline script to compute Jacobian matrices for Gemma-2-2b
* [ ]	Cloudflare Worker to serve per-word readouts
* [ ]	Wire real Gemma activations into the frontend
* [ ]	Pick the workspace layer band empirically for Gemma-2-2b (the paper found roughly the middle third of layers for a much bigger Claude model — Gemma-2-2b’s band will be narrower and needs to be found, not assumed)

Preference / conflict experiment

* [ ]	Conflict-prompt corpus for the preference/desire experiment
* [ ]	Contrastive direction computation (CAA)
* [ ]	Causal intervention test across held-out conflict types
* [ ]	Write up findings with explicit indicator-by-indicator honesty about what was and wasn’t shown

⸻

Known limitations of this whole approach

Worth stating up front rather than discovering from a critic later: the indicator-property method this project is built on is itself contested.

Critics argue that without a known, stable causal relationship between an indicator and consciousness, updating credence based on it is on shaky philosophical ground — not just under-evidenced, but structurally unable to close the gap it’s aimed at.

This project doesn’t resolve that debate. It aims to be rigorous enough that the only fair criticism left is that fundamental one, shared by every project of this kind, rather than any avoidable flaw in the method here.

⸻

Credits / reading

* Gurnee, Sofroniew, Lindsey, et al. — Verbalizable Representations Form a Global Workspace in Language Models (Anthropic, July 2026), transformer-circuits.pub
* Butlin, Long, Elmoznino, Bengio, Birch, Chalmers, et al. — Consciousness in Artificial Intelligence: Insights from the Science of Consciousness (2023), arXiv:2308.08708
* Rimsky et al. — Steering Llama 2 via Contrastive Activation Addition — method basis for the preference/desire experiment