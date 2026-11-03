# Where Should I Store API Keys for My AI Agents?

By [Mithilesh Gaurihar](https://www.linkedin.com/in/mithilesh-gaurihar/) · Last verified: October 2026

![Illustration on a dark background. Headline: Where should the key live? Below it, an open cream envelope labelled agent.pipe holds a card printed ${OPENAI_KEY}. A dotted line runs from the card to a brass key lying outside the envelope. Subtitle: The file holds the name. A scope holds the value.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/apikeys-01.png)

## Quick Answer

An API key should live somewhere your agent's config can point at by name, never in the config file itself. That rule holds whatever you build on. RocketRide Cloud implements it with three scopes: organization, team and user. The agent's pipeline references a name such as `${ROCKETRIDE_OPENAI_KEY}`, and the server fills in the value each time a run starts, so the file only ever holds the name. The scope you pick decides who can use the key, who can read it, and whether it still works after you deploy.

## Why Do API Keys End Up in Agent Config Files?

Because it's the fastest way to get an agent working. A typical agent project starts with a config file and a model call, and the API key goes wherever the code reads it first: pasted into the config, or into a `.env` file next to it.

From there, one `git add .` puts the key in the repository. It also travels with every copy of the project: a teammate's laptop, a bug report, a shared example. Rotating it means finding every copy, and nothing keeps a list of where they are. A stricter `.gitignore` only hides the problem; the fix is to keep the value out of the file entirely.

RocketRide pipelines are built for that. The file holds the variable's name, and the value lives in a scope on the server. The rest of this post covers how a key gets from a scope into a run, which scope to use, and what changes at deploy time.

## How Does a Key Get From a Scope Into a Run?

The pipeline names the variable instead of holding the value. This is the model node from RocketRide's own examples:

`"openai-4o": { "apikey": "${ROCKETRIDE_OPENAI_KEY}" }`

When a run starts, the server builds one set of variables from the scopes, substitutes every `${...}` in the pipeline, and hands the result to the run. Substitution happens on the server only; the clients send the pipeline with its placeholders intact. Three rules decide what a placeholder becomes:

| Placeholder | Result |
|---|---|
| `${ROCKETRIDE_OPENAI_KEY}`, variable set | Replaced with the value. |
| `${ROCKETRIDE_OPENAI_KEY}`, variable missing | Left as written. The node that needs it then stops with an error naming the missing variable and where to add it. |
| `${OPENAI_API_KEY}` (no prefix) | Replaced with `<REDACTED>`, and the run fails with an error saying only `ROCKETRIDE_` names resolve. |

The `ROCKETRIDE_` prefix is the security control. The source says why: any other name is blocked so a pipeline can't pull the server's own secrets out through `${VAR}` expansion. The named error for a missing variable matters too. Without it, the placeholder text would go to the provider as the key, and you'd only see the provider's "invalid API key" reply.

Two details keep the values short-lived. They're loaded at each start or restart, not kept on the task while it runs. And a pipeline with no placeholders loads nothing at all.

## Where Do API Keys Live in RocketRide Cloud?

On one Variables page, reached from the Cloud connection menu, with a list for each scope.

![Illustration of the RocketRide Cloud Variables page, headed One page, three scopes, one list each. Three cards stacked vertically. Organization variables, marked org admins only, holds ROCKETRIDE_SHARED_OPENAI_KEY. Team variables, marked team admins only, holds ROCKETRIDE_ANTHROPIC_KEY, ROCKETRIDE_GITHUB_TOKEN and ROCKETRIDE_SLACK_WEBHOOK. User variables, marked only you, in any scope you can run, holds ROCKETRIDE_OPENAI_KEY and ROCKETRIDE_SCRATCH_DB_URL. Every value is masked with dots and has Show and Delete buttons; each card has an Add Variable button. Caption line: values are stored encrypted and shown masked.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/apikeys-02.png)

*The Variables page, redrawn with placeholder names.*

Each row is a name and a value. Every name has to start with `ROCKETRIDE_`, and saving a scope is all or nothing: if one name in the batch is invalid, the whole save is refused, the stored set stays as it was, and the error lists the offending names. Values are encrypted at rest and shown masked.

The editor helps you use them. Typing `${` in any node field suggests the variable names your account has, and the editor warns before a run when the pipeline references a name your account doesn't have. That suggestion list carries names only, never values.

## Organization, Team or User: Which Scope Do I Use?

Each scope fits a different kind of key.

| Scope | Use it for | What it gives you | Can view the stored value | Can use it in a run |
|---|---|---|---|---|
| Organization | The shared account every team bills to | Set once, every team inherits it, rotate in one place | Org admins | Everyone in the org |
| Team | Credentials that belong to one product or squad | The team owns its keys, so people can join and leave without breaking the agent | Team and org admins | Everyone on the team |
| User | Your own key, for your own runs | Experiment against your own quota without touching what the team depends on | Only you | Only your own runs |

The rule I'd give a new teammate: **if a deployed agent needs the key, put it in the team scope.** The deploy section below explains why. Past that, put each key at the widest scope that's still true. A team credential copied into four people's user scopes is four things to rotate, with no list of where they are.

If you're on several teams, the team layer that applies to your interactive runs is the development team set on your profile. A team sent by a client is ignored on purpose, so nobody can choose whose credentials a run resolves under, and a run with no development team set is refused. When a teammate says a key is missing, ask which development team they have selected.

## Which Key Wins When Two Scopes Have the Same Name?

The narrower one. The layers stack from the server's own `.env`, through organization and team, to user, and each one overrides the last.

![Diagram titled Variable scopes: later layers override earlier ones; a team deployment stops one layer short, on purpose. Left column, Interactive run (you press Run in the editor): four stacked layers joined by arrows, numbered 1 Server .env (base, rarely used), 2 Organization (everyone in the org), 3 Team (everyone on the team), 4 User (only you). Right column, Team deployment (deployed to run on a schedule): the same first three layers, then a dashed red User box marked SKIPPED, not applied to a team-owned run. Footer: same name in two scopes, the lower band wins; a key that exists only in your user scope works locally and is absent after deploy.](https://raw.githubusercontent.com/mithileshgau/rocketride-blog-assets/main/images/apikeys-03.png)

*Interactive runs resolve four layers. A team deployment stops after the team layer.*

This is the feature you end up using most. A team points at the shared organization account while one developer tests against a personal key with its own rate limit, and neither edits a config file.

One more layer sits on top if you start runs from the Python or TypeScript SDK: both forward any `ROCKETRIDE_*` variables from your own shell, and those override all four scopes for that run. Nothing else in your environment is sent.

## Why Does My Agent Work Locally but Fail After Deploy?

Almost always because the key is in your user scope, and a team deployment doesn't read it.

An interactive run is yours, so it resolves all four layers. A team deployment belongs to the team, so it resolves the server `.env`, the organization and the team, then stops. The code says why: a team deployment's config must not depend on which person deployed it. If your user scope counted, the agent would carry your credentials into production and keep working until you changed teams. The server decides which kind of run it is; a client can't mark its own run as a deployment to change which layers apply.

There is a second version of this that catches people, and it has nothing to do with scopes. Both SDKs forward `ROCKETRIDE_*` variables from your own shell as the last layer, on top of everything the server merged. So a key exported in your terminal can make a local run work even when it is in no scope at all. A deployed run has no shell behind it, and the gap appears.

The fix for both is to move the key to the team scope and deploy again. It's annoying once, and after that no production agent depends on one person's account.

## Who Can See My API Keys?

Scoped and hidden are different claims, so here's exactly what each scope allows.

Your user variables are yours: the request always uses your own account, so there's no way to ask for someone else's. Organization values need org admin rights to read or write, and team values need team or org admin rights.

That covers the settings page, and the settings page is not the whole story. Everything a run resolves is readable by the code inside that run, including a tool node you installed from somewhere else. So a member who can run a pipeline in a scope can print the keys that scope supplies, whether or not the UI will show them. Admin rights control who can *view and edit* a stored value. They do not stop anyone whose runs resolve it from reading it. Treat a key in the organization scope as known to everyone who can run something there, and scope keys as narrowly as the work allows.

In RocketRide Cloud, each scope's variables are encrypted with Fernet, and the encryption key is held in the platform's secret store, not in the database. The server decrypts the variables and passes the resolved values to the run. The run never gets the encryption key, so code inside an agent, including a third-party tool node, can't decrypt the store.

Saves are recorded in an audit log by variable name, never by value, and a rejected save lists only names. Reading a value isn't logged, so clicking Show leaves no record.

## How Do I Rotate a Key Without Redeploying?

Update the variable, then restart the agent.

Values are read again at every start, including a restart, so there's no redeploy, no config change and no new version to publish. Rotating a provider key across every agent that uses it is one edit in one place, and each agent picks it up at its next restart.

## How Does This Compare With a .env File or a Secrets Manager?

|  | RocketRide Cloud scopes | RocketRide self-hosted | .env file in the project | Dedicated secrets manager |
|---|---|---|---|---|
| Where the value sits at rest | Encrypted per scope (Fernet) | Plain-text `.env` beside the server | Plain text on each developer's disk | Encrypted in the manager |
| Who can read it | Admins of that scope; only you for user scope | Anyone who can authenticate to the server | Anyone with the file | Whoever its access policies allow |
| Separate keys per team or person | Yes: organization, team, user | No, one shared list | One file per machine | Yes, by path or policy |
| How the agent gets it | Resolved into the pipeline at each start | Resolved into the pipeline at each start | Read by your code | Fetched by your code or an integration |
| Rotation | Edit the variable, restart | Edit the variable, restart | Edit every copy | Built in, often automatic |
| Read audit | No, writes only | No | No | Usually yes |

Scopes cover "the key shouldn't be in a file" and "my team needs this key". A dedicated manager adds short-lived credentials and an audit of every read, which matters for keys that gate customer data or money.

## What Should I Check in Any Tool That Holds My Agent's Keys?

- **Is the value ever in a file you commit?** The config should hold a reference, never the key.
- **Which variables can a pipeline reach?** An allowlist or prefix rule stops a pipeline from reading the host's own secrets.
- **What happens when a variable is missing?** A named error beats a confusing "invalid key" from the provider.
- **Does a deployed agent depend on one person?** Production should resolve from a shared scope.
- **Who can read values, and is that logged?** "Used by" and "readable by" should be separate lists.

## Where Does This Not Help Me?

**Scoping doesn't isolate a value from the run that uses it.** A team variable is hidden from people outside the team, but every value a run resolves is readable by the code in that run, including third-party tool nodes. Run processes don't inherit the server's environment, only an allowlist, but `ROCKETRIDE_` variables are on that allowlist by design.

**Nothing forces you to use the scopes.** Autocomplete and the pre-run warning are nudges. Paste a raw key into a node field and the editor shows it masked as `••••abcd`, but the full key is saved into the file. RocketRide's security docs say to treat such a file as a secret.

**There's no scope below user.** There's no per-agent secret: two agents needing different keys for the same provider need two variable names. I also found no expiry or version history for a stored value, so treat rotation as overwrite and restart, with nothing to roll back to.

**Self-hosted is simpler.** A self-hosted server has a single list instead of three scopes, stored as plain text in a `.env` beside the executable and readable by any authenticated client. That's fine for a single-tenant box you own, and it doesn't carry Cloud's guarantees.

The resolution, prefix and deploy rules are all in the open source [rocketride-server repository](https://github.com/rocketride-org/rocketride-server), if you'd rather read the code than take my word for it.

## FAQ

### Can a pipeline read the platform's own environment variables?

No. The bottom layer is whatever `ROCKETRIDE_*` variables the server process holds, so in principle a pipeline can reference those. Two things fence it off. The platform's own settings use a different prefix and are never a resolution layer at all, which the code states explicitly and applies to administrators too. And five `ROCKETRIDE_DB_*` names that carry database access are removed again before any pipeline process starts. What is left on that layer is whatever an operator deliberately put there.


### Are environment variables safe for storing API keys?

Environment variables are safer than a committed file, but the phrase covers two very different things. A variable held in a managed scope is encrypted at rest and resolved only when a run starts. A variable in a shell profile or a `.env` on disk is plain text that anything you run can read. The risk comes from where the value sits while nothing is running.

### What should I do if I already committed an API key?

If you committed an API key, revoke it at the provider first, before touching git. Rewriting history without revoking leaves a key that may already have been copied; revoking makes the leaked key useless whether or not you clean the history. Issue a replacement, put it in a scope rather than a file, and treat cleaning the history as tidying up.

### Can an AI agent leak the API keys it uses?

Yes, an AI agent can leak any key its own run resolves, which makes scope a security decision as well as an admin one. An agent that calls tool nodes you didn't write should get a narrow key with limited permissions at the provider, held in the team or user scope. The organization key everything else shares is the wrong one to hand to code you haven't read.

## Sources

- **Code:** [rocketride-server](https://github.com/rocketride-org/rocketride-server) on `develop` (commit 384e8300, October 26, 2026): substitution and the prefix rule in `packages/ai/src/ai/modules/task/pipeline.py`, the named errors in `packages/ai/src/ai/common/config.py`, layer merge and the team-deploy rule in `commands/cmd_task.py`, the subprocess allowlist in `task_engine.py`; RocketRide Cloud's service code for storage, encryption, permissions and auditing.
- **Docs:** [Security](https://github.com/rocketride-org/rocketride-server/blob/develop/docs/public/product/operate/security.md) and [Best practices](https://github.com/rocketride-org/rocketride-server/blob/develop/docs/public/product/guides/best-practices.md) in the RocketRide docs.
- **Background:** [Google: best practices for securely using API keys](https://support.google.com/googleapi/answer/6310037?hl=en-GB).
