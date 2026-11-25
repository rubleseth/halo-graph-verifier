# Halo Graph Verifier

Can one small Gemma model verify a proposed research graph claim in one generation, spend extra reasoning tokens when they help, and match RDG’s evidence quality at lower cost or latency? This repository is the planning workspace for the White Circle Halo competition. The study is in planning. The checkpoint, compute allocation, export coverage and performance remain open.

**Competition objective.** Prove the competition method on data exported from Research Data Graph (RDG) first. Transfer of the frozen model to a second crawler is a stretch goal. The competition window is December 1–10, 2026. Verify track-specific checkpoint, Halo version and submission requirements before selecting a recipe. The organizer requires public code and configurations for a submission. [Competition](https://whitecircle.com/halo/competition) · [Terms](https://whitecircle.com/halo/competition/terms) · [Halo](https://github.com/whitecircle/halo)

## Project ownership

RDG is a read-only source of exported crawl evidence and historical baseline results. Its ongoing repository, issues, configuration, services, data collection and graph are outside this project's write boundary. All data preparation, labeling, Halo SFT, replay, scoring and demo work belongs to this competition repository or its isolated environment. If the available exports lack a field, record the limitation here and adapt the competition experiment.

A future, separate fork of RDG is the intended integration target for Gemma as its reasoning engine. Creating the fork, running it and choosing its isolated graph and credentials are later decisions. This project sends no changes or writes back upstream.

## What the model decides

Start with RDG’s D08 question. Given a proposed graph claim and source evidence available at decision time, is that claim **supported**, **contradicted**, or **insufficiently evidenced**? A useful answer identifies the decisive source passage or the specific gap. For offline replay, the verifier reads a frozen packet and returns a proposed verdict. In a future fork, that fork’s controller would own retrieval, validation, identity resolution and writes to its own graph.

For example, a table reports a strength of 0.42 MPa. If a footnote assigns that value to the untreated control, a proposed link to the treated specimen is contradicted. If the packet contains only a paper title, the specimen and measurement remain unverified. These are illustrative examples.

## What makes it one call

The candidate receives one evidence packet once. Within one generation it can give a direct verdict or spend more tokens reconciling the evidence before giving its verdict. A wrapper that predicts a route and then starts a separate verifier call is a baseline. For supervision, run a frozen candidate twice offline on identical inputs, once in each forced mode, and independently score the resulting verdicts and evidence. Single-call adaptive behavior and savings remain hypotheses.

## Experiment design

1. Reconstruct each sampled decision using the permitted evidence available before inference. Preserve input and policy versions, source locators, group/family identifiers, stage, JEV outputs and their meanings, route, validator result, and resource use where reported. Separate subsequent evidence and reviewer corrections from the original input.
2. Sample ordinary accepts, abstentions, rejections and escalations across confidence ranges. Independently review the claims. Fit any thresholds on development data. Extra reasoning cannot recover a missing source. Probability scores and routing history alone establish neither truth nor ideal Gemma behavior.
3. Use direct and thinking replays to identify rescues, regressions, ties and failures in both modes. Decide how those outcomes supervise an adaptive generation. Derive synthetic matched pairs from observed failure modes. Reserve independent real dataset families and later traffic for evaluation. Keep synthetic siblings and repeated sightings grouped.
4. Compare always-direct, always-thinking, adaptive Gemma and the existing route using recorded RDG outputs or a separately executed baseline on the same inputs. Count accepted unsupported claims, supported claims recovered, evidence quality, model calls, total tokens, end-to-end latency and cost per 1,000 decisions. Measure the real traffic mix and account for idle capacity for an always-on host.
5. Demonstrate what a querying agent gains from verified links, source locators and explicitly unresolved evidence. The transfer test freezes the checkpoint before evaluating another crawler through a new input adapter.

## Planning and evidence

The opening decision map is in [Wayfinder](docs/WAYFINDER.md). The [glossary](GLOSSARY.md) defines the terms shared across sessions. The [starter archive](references/ticket-verifier-starter-v1.zip) preserves the earlier ticket-completion exercise, scorer, generator and two-run labeling script. Its 762 synthetic training rows and 249 evaluation rows are teaching material. The classifier’s development result applies only to that exercise. D08 needs its own corpus, labels, and evaluation.

The first useful artifact for a training decision is an independently reviewed, replayable batch of D08 decisions. If live traffic is insufficient, document the sample actually available and the narrower conclusion it supports. Publish only data permitted for redistribution, or supply a reproducible public fixture and retrieval procedure. [Halo SFT](https://whitecircle.com/halo/docs/training-methods/sft) · [Gemma 4](https://deepmind.google/models/gemma/gemma-4/) · [Hybrid reasoning precedent](https://arxiv.org/abs/2505.14631)
