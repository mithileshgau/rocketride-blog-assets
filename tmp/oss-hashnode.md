> Originally published on [dev.to](https://dev.to/mithilesh_gaurihar/is-open-source-ai-still-open-in-2026-pfi).

By [Mithilesh Gaurihar](https://www.linkedin.com/in/mithilesh-gaurihar/), who builds RocketRide and writes about the runtime and Cloud infrastructure behind production AI applications.

**Update log.** Published September 2, 2026. At that date the Nvidia and Hugging Face deal was reported but confirmed by neither company. This line gets updated when that resolves.

> **Disclosure:** We build RocketRide, an open source AI pipeline runtime, which is one of the tools this post talks about. This is not an independent assessment of the acquisitions described below. Every external claim is linked to its source, and the post says which claims rest on reporting that no company has confirmed.

## Quick Answer

**Partly.** Open source AI in 2026 is still open by licence and consolidating by ownership: no model was relicensed this year, but between July 1 and September 1 the distribution layer went into play, the routing layer was sold, and compute repriced. The test that survives an acquisition is not what licence the weights carry, it is how many independent companies would have to agree before your access could change, counted across five layers: licence, distribution, runtime, compute, orchestration.

Most teams score well on the first layer and badly on the other four, because the licence is the only one that announces itself. RocketRide is at the orchestration layer, MIT licensed and self-hostable, and it makes provider choice a configuration field rather than a code path, so moving a pipeline to a different provider or a self-hosted endpoint does not require rewriting it. That covers one layer of the five. It does nothing about who owns the chips or where the weights are hosted, and our own project governance is company-led rather than held by a foundation, a weaker guarantee than the one DuckDB demonstrated in August.

## What Changed in Open Source AI in 2026?

Two acquisitions inside the AI stack, six weeks apart, at two different layers.

Stripe's acquisition of OpenRouter is the settled one, [announced by Stripe](https://stripe.com/newsroom/news/stripe-agrees-to-acquire-openrouter) and confirmed by both parties, [reported at more than $7 billion](https://techcrunch.com/2026/08/16/stripe-will-reportedly-acquire-ai-gateway-startup-openrouter-for-7b/). OpenRouter routes across more than 400 models from more than 80 providers, and teams adopted it specifically to stay provider-neutral. The stated plan is continuity: same name, same product, same roadmap.

The distribution layer is in play rather than sold. The Information reported on August 26 that Nvidia had agreed to buy Hugging Face for about $12.9 billion, and [TechCrunch covered it the same day](https://techcrunch.com/2026/08/26/nvidia-closes-in-on-hugging-face-acquisition/), noting that neither company had confirmed anything and that Business Insider understood the talks had not produced a signed agreement. Treat it as reported, because that is what it is. What makes it worth watching anyway is a detail that predates it: the ggml and llama.cpp team, whose runtime serves a large share of local inference, [joined Hugging Face in February 2026](https://huggingface.co/blog/ggml-joins-hf). Should the deal close, distribution and one of the dominant local runtimes land under the same owner as the chips, without either layer being bought directly.

There is an obvious rebuttal here, and it deserves a straight answer. Nvidia sells hardware, so a healthy open ecosystem that runs on that hardware is worth more to it than a closed one. That is true, and it is the reason to expect no dramatic change. It is also an incentive rather than a commitment, and incentives get revised when the business does. The same company's compute partnership programme, announced July 1, is the useful illustration: providers would keep a base hourly rate and hand over half the revenue above it, and they were expected to vet customers through an approval process. That is a gate. It was built by a company with every commercial reason not to want one, which tells you what the structural answer is worth compared to the incentive argument. Reporting in late August said the programme had been [paused](https://finance.yahoo.com/technology/ai/articles/nvidia-pauses-ai-cloud-revenue-120700044.html) after partner pushback and internal antitrust concerns, and [Nvidia denies pausing it](https://www.tomshardware.com/tech-industry/artificial-intelligence/nvidia-denies-pausing-ai-cloud-commitments-initiative-after-reported-partner-backlash-report-claims-company-told-cloud-providers-it-could-only-lease-its-gpus-to-nvidia-approved-customers). Either way, the terms were drafted.

None of this changed a licence. Every MIT-licensed model is still MIT licensed, which is why the licence is the wrong thing to check.

![A diff of the AI stack, before and after August 2026. One unchanged line: licence, still open. Three pairs of changed lines. Distribution goes from an independent repository to an acquisition reported on August 26. The local runtime goes from an independent project to the same owner as distribution. Orchestration goes from provider-neutral by design to acquired on August 16. Every changed line is an ownership change rather than a licence change.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/oss-01-what-changed.png)

## What Is the Difference Between Open Weights and Open Source?

Open weights means the model file is published under some licence. Open source in the fuller sense also means governance: who decides the project's direction, and whether that decision-making survives a change of ownership.

The clearest demonstration this year came from outside the AI stack entirely. AWS [acquired DuckLabs](https://aws.amazon.com/blogs/big-data/aws-and-ducklabs-building-the-future-of-analytics-together/), the company behind DuckDB, announcing on August 26 and [closing on August 31](https://www.theregister.com/databases/2026/08/26/aws-buys-ducklabs-the-people-behind-the-popular-in-process-olap-database/). The transaction did not include the open source project, which stays with the nonprofit DuckDB Foundation under an MIT licence, with the original creators continuing to lead technical direction. The acquisition went through and the project's ownership held, because someone had built a structure years earlier that kept the two separable. That is what durable openness looks like, and it is a property of the structure rather than of the licence.

The licence-level version of the same lesson arrived the same week. Z.ai released [GLM-5.3-Flash](https://huggingface.co/zai-org/GLM-5.3-Flash) on August 26 under an unmodified MIT licence. Two days later it released the larger GLM-5.3 under [a bespoke licence carrying a model-as-a-service revenue gate](https://thenewstack.io/zai-glm-weights-license/): operators above $10 billion in aggregate revenue must pass a security review whose scope and method Z.ai determines. Same lab, same family, two days apart, two different sets of terms. Alibaba's [Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next), released open-weight on the same day, ships under the Qwen Community License rather than MIT. "Open" has become a per-artifact property, and each artifact has to be read on its own.

Debian settled its own version of the question through governance. On August 29 the project passed a [general resolution on responsible use of generative AI](https://www.debian.org/vote/2026/vote_002), after a two-week vote across nine options. It neither endorses nor prohibits the tools, holds AI-assisted contributions to the same standards of quality, correctness and maintainability as any other, and encourages disclosure without requiring it. Whatever you make of the outcome, the mechanism is the point: a body with standing decided, and that decision outlives whoever shows up next.

## How Do I Audit Whether My AI Stack Is Really Open?

There are five layers between you and a running model, and you own none of them outright. The **licence** is the terms on the weights. **Distribution** is where you obtain them. The **runtime** is what loads and serves them. **Compute** is the chips and the machines holding them. **Orchestration** is what calls the model and routes between providers.

Go layer by layer and ask one question of each. If the owner changed the terms tomorrow, what breaks, and how long would it take to move? Then count the layers where the answer is "one company decides".

![The five-layer audit as a diff, headed "lines most audits are missing". One unmarked line, licence, is checked and passes. Four lines are marked as needing to be added to the audit: distribution, where you obtain the weights; runtime, what loads and serves them; compute, who prices the machines; orchestration, what routes the call. The question to put to each is what breaks if that layer's owner changed the terms tomorrow.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/oss-02-layer-audit.png)

Compute is where this stops being abstract. Nvidia customers were told in August that [AI server prices are rising more than 15 percent](https://fortune.com/2026/08/22/nvidia-customers-ai-related-price-hikes-15-percent-vera-rubin-grace-blackwell-chips/) at the system level, driven by memory costs and landing on systems shipped early next year. Server makers put the increase on the chips themselves at around 17 percent, varying with memory configuration. Those are two different figures measuring two different things, and both are terms changing at a layer almost nobody audits, on a timescale you do not control.

Run the audit on your own repository too, because that is where it produces surprises. Doing it on ours turned up `modelSource: "openrouter"` sitting in the provider node definitions, 346 occurrences across thirteen files. It isn't a routing path, it's the recorded provenance of the context-window numbers we publish for each model, with the comment `// openrouter` next to the token limits it sourced. Harmless, and still a dependency on a company that had just been acquired, in a place nobody put it deliberately. The layers you did not choose are the ones the audit is for.

## How Do I Reduce the Number of Companies My Stack Depends On?

Four moves, ordered by what they return relative to the effort.

**1. Hold your own copy of the weights.** If a model matters to production, don't depend on pulling it at deploy time. Mirror it where you control the storage. This is the cheapest item on the list and the one most often skipped.

**2. Keep the serving path replaceable.** Prefer models more than one runtime can serve. GLM-5.3-Flash lists SGLang, vLLM, Transformers and KTransformers on its model card; Qwen3.8-Flash-Next lists vLLM, SGLang and Transformers. Two independent serving stacks put you in a meaningfully different position from one, and you can check which stacks a model supports before you commit to it.

**3. Keep model choice in configuration, not in code.** The teams absorbing this year's changes most easily are the ones for whom switching is a settings change. AT&T [reports](https://about.att.com/blogs/2026/the-tokenomics-equation.html) routing about 40 percent of employee AI usage to open models today, an intent to reach 60 to 70 percent, coding costs down 56 percent against roughly a 2 percent quality decline, at around 45 billion tokens a day. Those figures are AT&T's own and have not been independently audited, which in a post arguing against taking vendors at their word is worth saying out loud. The shape of the claim is still the useful part: that kind of migration is only practical when routing is a configuration decision.

**4. Make sure the orchestration layer is one you could run yourself.** This is the layer that consolidated most recently and the one fewest teams have audited. The question to ask of any orchestration tool: if this company were acquired tomorrow, could we keep running it on our own infrastructure, from source, without asking anyone? Plenty of tools pass. LiteLLM, which is what AT&T routes through, is MIT licensed across the majority of its code and self-hostable, with a paid enterprise tier on top, and that combination is a perfectly good answer to the question. The point of the test is that it has a yes-or-no answer you can check, not that it selects for any particular tool.

## Where Does RocketRide Fit, and Where Does It Stop?

We are the orchestration layer, and that last question is the one we're built to answer with yes. The runtime is MIT licensed, it runs on your own infrastructure, and a pipeline is portable JSON rather than a hosted object.

Concretely, provider choice is a configuration field. The OpenAI-compatible node takes `base_url` and `model` as config properties, so pointing a pipeline at Together, Groq, a vLLM deployment, LM Studio or an Ollama instance means changing two fields rather than writing a new code path. The branded presets in [the repository](https://github.com/rocketride-org/rocketride-server) are the same implementation with the base URL pinned. That is the portability claim, and it is small enough to verify in the source in about a minute, which is the point of stating it that narrowly.

![A diff of a pipeline definition, headed "two fields, one pipeline definition". The node line is unchanged: openai-compatible. Two fields change. The base URL moves from a hosted provider endpoint to a self-hosted vLLM address, and the model moves from provider-hosted to open-weight and self-served. The pipeline line below is unchanged. Moving providers changes two configuration fields and nothing else, and it covers one layer of five: the chips and the weights still belong to someone else.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/oss-03-portability.png)

Being honest about scope: this covers one layer of five, and specifically not the runtime layer in the sense this post uses the word. What loads and serves the weights is still vLLM or SGLang or llama.cpp, and we do not replace any of them. What we do is keep which one you point at a matter of configuration. We change nothing about who owns the chips, and nothing about where the weights are hosted. One company comes off the list of parties who would have to agree before your stack keeps working, which is worth doing precisely because it is a layer you can still choose.

Our own governance is the weaker kind, by the standard this post argues for. RocketRide's [governance document](https://github.com/rocketride-org/rocketride-server/blob/develop/GOVERNANCE.md) defines contributor, reviewer and maintainer roles, requires two maintainer approvals for significant changes, and resolves disputes by maintainer consensus, with the project lead making the final call if consensus fails. That beats no published process. It is company-led, not foundation-held, and DuckDB's structure is stronger than ours. The MIT grant on every release you already have is irrevocable, and a fork is always available. Those are real protections. They are not equivalent to a governance body that outlives the company, and we should not pretend otherwise while making this argument.

## What Should I Do First?

Open source AI in 2026 has not become less open by licence. What has happened is a concentration of ownership around it, and the two call for different remedies. A licence audit will pass while your actual dependency count quietly climbs.

So run the count. Five layers, one question each, and then rehearse the move you claim you could make, on a deadline, in staging, before you need it. Start with the layer where recovery would take longest, which for most teams is orchestration, because it touches every pipeline. Where the answer is that one company decides and you have no rehearsed alternative, you have found the layer that fails first.

Then run the same audit on us. The `base_url` field, the governance document and the node definitions are all in [rocketride-server](https://github.com/rocketride-org/rocketride-server) under MIT, so none of the claims above need taking on trust. Self-host when owning the infrastructure is part of the job, and use [Cloud](https://cloud.rocketride.ai/) when the pipeline is the work and operating the platform is not; either way the artifact is yours. If you run the audit and find a layer we missed, we're on [Discord](https://discord.gg/PMXrtenMsY).

## FAQ

### Does an MIT licence mean a model is safe to depend on?

No. An MIT licence means you're permitted to use, modify and redistribute the weights. It does not guarantee you can still obtain the file, that a runtime will still serve it, or that you can afford the hardware underneath it. Those are three separate layers with three separate owners, and the licence governs none of them.

### Is open source AI dead in 2026?

No. Licences are unchanged, and August alone brought two significant open-weight releases: Z.ai's GLM-5.3-Flash under MIT, and Alibaba's Qwen3.8-Flash-Next under the Qwen Community License. What changed is that several layers around the model consolidated: distribution went into play, the dominant local runtime moved with it, compute repriced, and provider routing was sold, all between July and September 2026.

### Does self-hosting solve this?

Partly. Self-hosting is available at two of the five layers: you can run your own model server, and you can run your own orchestrator, and each is a separate decision with its own tool. It does not address who manufactures and prices the chips, and it does not address where the weights are hosted for you to download in the first place. It reduces the count rather than eliminating it.

### Did the Nvidia and Hugging Face deal change any model licences?

No, and that is the point. As of September 2, 2026 the deal is reported rather than confirmed by either company, and no licence on any model has changed as a result. What would change is who owns the repository most open models are distributed through, and who owns the ggml and llama.cpp team that joined Hugging Face in February 2026.

## Sources

Claims resting on reporting rather than a company statement are marked as such in the body. Links above go to the source used for each claim, and to a primary source wherever one exists.

- The Information, "Nvidia Agrees to Buy Open Source AI Platform Hugging Face For $12.9 Billion", August 26, 2026 (reported; unconfirmed by either company)
- TechCrunch, ["Nvidia closes in on Hugging Face acquisition"](https://techcrunch.com/2026/08/26/nvidia-closes-in-on-hugging-face-acquisition/), August 26, 2026
- Hugging Face, ["GGML and llama.cpp join HF to ensure the long-term progress of Local AI"](https://huggingface.co/blog/ggml-joins-hf), February 20, 2026
- Stripe, ["Stripe agrees to acquire OpenRouter"](https://stripe.com/newsroom/news/stripe-agrees-to-acquire-openrouter)
- TechCrunch, ["Stripe will reportedly acquire AI gateway startup OpenRouter for $7B+"](https://techcrunch.com/2026/08/16/stripe-will-reportedly-acquire-ai-gateway-startup-openrouter-for-7b/), August 16, 2026
- AWS, ["AWS and DuckLabs: Building the future of analytics together"](https://aws.amazon.com/blogs/big-data/aws-and-ducklabs-building-the-future-of-analytics-together/), August 26, 2026
- The Register, ["AWS buys DuckLabs, the people behind the popular in-process OLAP database"](https://www.theregister.com/databases/2026/08/26/aws-buys-ducklabs-the-people-behind-the-popular-in-process-olap-database/), August 26, 2026
- Fortune, ["Nvidia customers notified about AI-related price hikes above 15%"](https://fortune.com/2026/08/22/nvidia-customers-ai-related-price-hikes-15-percent-vera-rubin-grace-blackwell-chips/), August 22, 2026 (reported)
- Yahoo Finance, ["Nvidia pauses AI cloud revenue-sharing deals over antitrust concerns"](https://finance.yahoo.com/technology/ai/articles/nvidia-pauses-ai-cloud-revenue-120700044.html) (reported)
- Tom's Hardware, ["Nvidia denies pausing AI cloud commitments initiative after reported partner backlash"](https://www.tomshardware.com/tech-industry/artificial-intelligence/nvidia-denies-pausing-ai-cloud-commitments-initiative-after-reported-partner-backlash-report-claims-company-told-cloud-providers-it-could-only-lease-its-gpus-to-nvidia-approved-customers)
- Debian, ["General Resolution: LLM usage in Debian"](https://www.debian.org/vote/2026/vote_002), result announced August 29, 2026
- LWN.net, ["Debian votes to allow 'responsible use of generative AI'"](https://lwn.net/Articles/1091231/)
- Hugging Face, ["zai-org/GLM-5.3-Flash"](https://huggingface.co/zai-org/GLM-5.3-Flash) model card, MIT licence, released August 26, 2026
- The New Stack, ["Z.ai's GLM-5.3 goes open weight, but its new license aims at hyperscalers"](https://thenewstack.io/zai-glm-weights-license/)
- Hugging Face, ["Qwen/Qwen3.8-Flash-Next"](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) model card, Qwen Community License 1.0, released August 26, 2026
- AT&T, ["The Tokenomics Equation: Balancing Cost and Performance"](https://about.att.com/blogs/2026/the-tokenomics-equation.html) (self-reported, not independently audited)
- Fierce Network, ["Open models are driving AT&T's AI 'tokenomics' strategy"](https://www.fierce-network.com/cloud/open-models-are-driving-atts-ai-tokenomics-strategy)
