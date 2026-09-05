# gemma-jlens

Recreating Anthropic's J-space / Jacobian lens technique on Gemma-2-2b,
with a chunky-toy visualizer of concept activations.

## What is this?

In July 2026, Anthropic published *"Verbalizable Representations Form a
Global Workspace in Language Models,"* introducing a technique called the
**Jacobian lens (J-lens)**. It identifies a subset of a model's internal
activations — the **J-space** — that behaves like a "global workspace":
a small, privileged set of concepts the model can report on, hold in
mind, reason with, and flexibly reuse, sitting on top of a much larger
volume of automatic processing it can't access at all.

This project recreates that technique on **Gemma-2-2b** (open-weight,
so fully inspectable) and wraps it in a playful, "chunky sketch toy"
interface: type a word, watch which concepts light up in the model's
internal workspace as it moves through the network's layers.

This sits alongside my other Gemma interpretability work — see
[`gemma-scope-player`](https://github.com/geminigooner/gemma-scope-player),
which sonifies SAE feature activations instead of visualizing J-lens
readouts.

## How the technique works (short version)

- Every layer of the model has a **residual stream** — a running vector
  that gets progressively enriched as it passes through the network.
- The J-lens estimates, per layer, the *average* causal effect that
  nudging the residual stream at that layer has on the model's eventual
  output — averaged across hundreds/thousands of unrelated prompts, so
  it captures a concept's *general disposition to be said*, not just
  what's happening in one sentence.
- That gives one matrix per layer. Multiply any activation by its
  layer's matrix, run it through the normal output layer, and you get a
  ranked list of words that activation is "pointed toward."
- Collect those readouts across all layers for a single input word, and
  you get a picture of what the model is "thinking about" as it
  processes that word — including concepts that never appear in the
  input or output at all.

## Architecture

Two pieces that never share a codebase or a deploy target, on purpose
(lesson learned from past projects getting tangled):

```
/offline/     one-time Python script: computes & saves per-layer
              Jacobian matrices from a background text corpus.
              Runs once, offline, not part of the live app.

/worker/      Cloudflare Worker: loads the saved matrices, takes a
              word, returns per-layer top concepts as JSON.

/frontend/    the actual toy — type a word, drag a layer slider,
              watch concept-blobs bloom and fade. Static, talks
              only to the Worker's JSON endpoint.
```

## Status

- [x] Design mockup with placeholder data (`gemma-thinks-mockup.jsx`) —
      nailing down the "chunky sketch toy" visual language before
      wiring in real output
- [ ] Offline script to compute Jacobian matrices for Gemma-2-2b
- [ ] Cloudflare Worker to serve per-word readouts
- [ ] Wire real Gemma activations into the frontend
- [ ] Pick the workspace layer band empirically for Gemma-2-2b (the
      paper found roughly the middle third of layers for a much bigger
      Claude model — Gemma-2-2b's band will be narrower and needs to be
      found, not assumed)

## Credits / reading

- Gurnee, Sofroniew, Lindsey, et al. — *Verbalizable Representations
  Form a Global Workspace in Language Models* (Anthropic, July 2026),
  [transformer-circuits.pub](https://transformer-circuits.pub/2026/workspace/)
