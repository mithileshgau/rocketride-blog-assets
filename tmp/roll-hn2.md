By Mithilesh Gaurihar. Last verified: September 2026.

## Quick Answer

Versioning an AI pipeline in production means splitting one release into two separate acts: building an immutable numbered version (v1, v2, v3) that serves no traffic, and pointing an environment at one of those versions. Because every version stays built and unmodified, rolling back is a pointer move rather than a rebuild, so recovery costs a request instead of a deployment cycle. In RocketRide these two acts are called deploy and publish: deploy snapshots the current build as an immutable version, publish points an audience at one. Any system that keeps old versions addressable can work this way, and the four-point checklist below applies to whatever tool you already use.

## The failure that makes this urgent

One prompt changed: a few words in an extraction step, shipped on a Tuesday, reviewed by one person, obviously fine.

Three weeks later a quarter of your documents come back wrong. They are not broken, which you would have caught on day one. They are wrong in a way that took three weeks of downstream use to surface.

You know which version was good. The problem is what going back to it costs: a full rebuild, a fresh push, a cold start on the far side, and the broken version answering every request while you wait. The fix costs an afternoon you did not plan for.

The prompt change was always going to happen. What is worth engineering away is the asymmetry: a two-minute edit that takes an afternoon to undo. Nobody would accept this for ordinary software, but the AI parts arrived with their own habits: prompts drifting a word at a time, pipelines edited live in production, no named version to go back to.

## Why is model versioning not pipeline versioning?

Search this question and the answers are about models. They are good answers to a different question.

[Ayinedjimi Consultants](https://dev.to/ayinedjimi-consultants/ai-model-versioning-and-rollback-strategies-for-production-2hfe) covers checkpoint strategy and rollback triggers, [Sysart](https://sysart.consulting/insights/automated-model-rollback-on-premises-ai/) automated rollback on-premises, [Sandgarden](https://www.sandgarden.com/learn/rollback) rollback as an operational discipline, and [BuildMVPFast](https://www.buildmvpfast.com/blog/agent-versioning-rollback-production-ai-update-zero-downtime-2026) zero-downtime agent updates. All of them are about weights, checkpoints, data hashes and feature flags, and on that subject they are right.

The gap is that in a production AI system the model is one field in a much larger artifact. What changed on that Tuesday was a prompt, not a weight file. Alongside the prompt sit the graph of nodes and their wiring, the retrieval config, the tool definitions, the output schema and the branch conditions. A model registry versions none of that.

One of those sources concedes the point better than I could put it: "Rolling back to the previous checkpoint restores the model weights, but it does not restore the production context in which those weights were validated."

That production context is the pipeline. If it is not itself a versioned artifact, restoring it means reassembling it, and reassembly is the expensive step in the story above.

## The three approaches

Most teams use one of these. They are not exclusive, and the third does not replace the first two.

| Approach | What it versions | How I roll back | What it does not cover |
| --- | --- | --- | --- |
| Feature flags | The decision to run new code or old, at a branch point written in advance | Flip the flag, effective in seconds | Only paths that were predicted. A prompt edit inside an unflagged node is invisible to it, and flags accumulate into their own maintenance problem |
| Model registry plus checkpoints | Weights, training data hashes, evaluation results, model lineage | Re-point the serving layer at an earlier checkpoint, then redeploy the surrounding app | The graph, the prompts, the retrieval config, the wiring. Restores the model, not the context it was validated in |
| Immutable pipeline versions | The whole pipeline artifact: graph, prompts, node config, retrieval settings, wiring, as one numbered version | Publish the previous version to the affected audience; it is already built | Anything outside the pipeline boundary. Provider behaviour, your data, and writes already committed are all untouched |

RocketRide sits in the third row only. It is not a model registry; if you train your own models you still want [MLflow](https://mlflow.org/) or [Weights & Biases](https://wandb.ai/) alongside it, versioning what it does not.

![Two columns on a dark ground. On the left, headed DEPLOY, creates a version and serves nothing, three cards labelled v1, v2 and v3, each marked sha256 verified. On the right, headed PUBLISH, binds one audience to one version, three rows labelled user, team and public, with user and team pointing at v3 and public pointing at v2. Connectors run from each version to the audiences bound to it. Audiences are independent, so user, team and public can sit on different versions.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/roll-01-deploy-publish.png)

In RocketRide, deploy snapshots the current build as the next immutable version in the org registry and serves nothing to it. Publish points one audience at one of those versions, and it is the same act whether you are releasing, promoting or rolling back. Only the target version differs.

The audiences are personal (`@me`), a team, and the public store. Each artifact is stored under a version number with its sha256, never overwritten and re-verified on load, so what runs is provably the bytes you deployed. Every version and publish lands in an audit history.

**Two limits.** Audience-level publishing applies to apps rather than raw pipelines today, so for a bare pipeline the unit is the team pointer. And the public store is review-gated: internal audiences serve instantly, but a store version waits for approval. Rolling the store back to an already-approved version is still a pointer move; rolling *forward* is not.

![Three version cards on a dark ground labelled v1, v2 and v3, each marked still built, with v2 ringed. A dashed pointer leaves v3, drops below the row and turns back to v2, labelled publish to v2. A note reads: the bad version stays deployed and unmodified, it just stops being served. The caption reads: v2 has been sitting there built since the day it was deployed.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/roll-02-rollback.png)

The rollback is the same publish act aimed at an earlier version number. The bad version stays in the registry unmodified, it simply stops being pointed at, so you can put it in front of a staging team to reproduce the failure while production sits on the known-good version.


On startup cost: the artifact is read and sha256-verified on load, so a rolled-back version is not free, it is just not a rebuild. The saving is the build and push cycle, not every millisecond.

## What good looks like

Four questions. Ask them of whatever you already run, not just of us. If the answer to any is no, that is where your next incident will cost more than it should.

1. **Can I name the exact version serving traffic right now?** Not the branch, not the last commit anyone remembers pushing. A version identifier readable off the running system, for each audience separately.
2. **Can I point traffic at a previous version without a rebuild?** If the honest answer involves a CI run, recovery time is build time, and the bad version is live for all of it.
3. **Is the old version still there, unmodified?** Being able to reconstruct it from git is not the same as it sitting there built. Reconstruction is the step that fails at 2am.
4. **Can I replay a specific past run?** Knowing which version was live is half the question. The other half is what that version actually did on the request that went wrong. In RocketRide, deploy runs are recorded to a per-task run log (`run_log.py`), so a past run can be replayed rather than reconstructed from log lines.

Plenty of setups pass these without buying anything. If you ship pipelines as immutable container images with a pinned tag and traffic is a load balancer pointer you can move, you already have most of this. The point is knowing which ones you pass before you need them.

## Where this does not help me

Pipeline versioning solves one class of problem. Here is what it leaves entirely alone.

**A model provider changing behaviour underneath you.** Your artifact is byte-identical and the hosted model behind it is not the one you validated against. Rolling back changes nothing, because the pipeline was never what moved. Pinning model versions where your provider offers them helps more than any rollback will.

**Data drift.** If your inputs have shifted, v2 will handle them as badly as v3 did, because it was validated on data that no longer arrives. Rolling back restores older assumptions about the world, not correctness.

**Writes your pipeline already made.** Repointing does not un-write rows, un-send emails or un-call webhooks. Everything the bad version did downstream is still done, and cleaning it up is a data problem with no version number attached.

**A bad version that ran for three weeks.** This is the honest one, because it is the scenario this post opens with. There are two gaps in that story, and versioning only closes one of them.

The gap from **ship to notice** was three weeks. The gap from **notice to fix** was an afternoon. Immutable versions collapse the second gap to a pointer move and do nothing at all about the first. Evaluation and monitoring are what close the gap that actually cost you the quarter, and if you only have budget for one of the two, it is not this one.

## FAQ

### What is the difference between deploying and publishing a pipeline?

Deploying builds an immutable numbered version of the pipeline and binds nothing to it, so no traffic is served as a result of deploying. Publishing points one audience at one version. Keeping them separate is what makes rollback cheap: the versions already exist, so changing which one is live is a pointer move rather than a build.

### How is rolling back an AI pipeline different from rolling back code?

Mechanically it is the same idea, and that is the point: application deployment solved this years ago with immutable artifacts and a pointer. AI pipelines mostly did not inherit it, because the tooling grew up around model checkpoints. The difference is what sits inside the artifact: a pipeline version carries prompts, retrieval config and node wiring, not just a binary.

### Do I need a model registry if I version pipelines?

If you train or fine-tune your own models, yes. A pipeline version records which model it calls; it does not version the weights, training data or evaluation runs behind them. MLflow or Weights & Biases cover the half that pipeline versioning does not.

### Can I roll back a prompt change without redeploying?

Yes, if the prompt is part of the versioned artifact rather than a value read at runtime. The version containing the old prompt is still built, so pointing at it restores that prompt with no rebuild. Prompts living in a database or an environment variable sit outside the artifact and need their own history.

### What happens to my in-flight runs when I roll back?

Repointing changes which version new requests get. It does not reach into work already running, so a run that started on the bad version finishes on it. Expect a short window where both versions produced output, and drain before repointing if that matters for correctness.

## Sources

- **Source material:** RocketRide newsletter, [Mission 17](https://news.rocketride.ai/mission-17/) and [Mission 13](https://news.rocketride.ai/mission-13/)
- **Product:** [RocketRide Cloud](https://cloud.rocketride.ai/), [rocketride-server](https://github.com/rocketride-org/rocketride-server) (MIT), [docs](https://docs.rocketride.org/)
- **Prior art on model versioning:** [Ayinedjimi Consultants](https://dev.to/ayinedjimi-consultants/ai-model-versioning-and-rollback-strategies-for-production-2hfe), [Sysart](https://sysart.consulting/insights/automated-model-rollback-on-premises-ai/), [BuildMVPFast](https://www.buildmvpfast.com/blog/agent-versioning-rollback-production-ai-update-zero-downtime-2026), [Sandgarden](https://www.sandgarden.com/learn/rollback), [arXiv 2508.11824](https://arxiv.org/pdf/2508.11824)
