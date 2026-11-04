# An open-source candidate sourcing agent for Kortix

Kortix is open source, and this repository is the candidate sourcing agent you can read, fork and change before you let it run on a cron. It searches LinkedIn for people who match one open role, then checks that role's Greenhouse pipeline so nobody already in it gets a second cold email. One outreach message per new match waits for a recruiter to review it, and a run ends at the review inbox. Nothing sends. This is the agent Kortix runs on its own outbound pipeline. It publishes the files that make it work: the prompt, the skill, the manifest, and three documents behind the decisions. It runs on Kortix, the open-source CompanyOS.

## What one run leaves behind

From [Kortix's write-up of the run](https://kortix.com/use-cases/candidate-sourcing):

> Reviewed **31** LinkedIn profiles and dropped **13** that were already in the pipeline. Drafted **18** personalized emails, each with the candidate's current role and the reason they matched. All **18** are in the review inbox, and nothing was sent.

Thirteen of thirty-one is the number worth staring at. Forty-two percent of the people a good search surfaced were already known to the company. Most of the work in a trustworthy sweep happens before an email is written.

## The four rules that stop a run

The guardrails are quoted as written, and each one is checkable. Two are enforced twice. The prompt states them in its own words, and the connector policies in `kortix.yaml` allow a pipeline read while every write on either connector waits for a person. The third is a test any reviewer can apply in ten seconds. The fourth caps a bad search at twenty drafts.

- "The agent never sends outreach itself and never adds a candidate to Greenhouse."
- "The agent checks the role's Greenhouse pipeline before drafting anything."
- "If the draft reads the same without them, rewrite it."
- "Draft at most 20 candidates per run."

## What is in the repository

| Path | What it holds |
| --- | --- |
| `kortix.yaml` | The agent block, the skill, both connectors with their policies, the cron trigger |
| `agents/candidate-sourcing.md` | The system prompt: the job, the order of steps, the output |
| `skills/outbound-sourcing/SKILL.md` | The numbered rules a run follows, including the dedupe step |
| `docs/how-a-run-works.md` | One run from trigger to review inbox, with the 31/13/18 example |
| `docs/dedupe-rules.md` | What counts as already contacted and what a skip looks like |
| `docs/what-it-is-not.md` | The boundaries, and when to buy a product instead |

## Run it on a Kortix project

An agent is one `agents.<name>` block in `kortix.yaml`: its behavior (`prompt` or `prompt_file`, `model`, `permission`) and the registered skills, tools, commands and extensions it enables. `agents/<name>.md` holds the prompt alone. The manifest here is complete, so a clone needs only a login and a ship:

```bash
git clone https://github.com/Kortix-seo/candidate-sourcing-agent
cd candidate-sourcing-agent
kortix login
kortix ship
```

`kortix ship` validates the manifest, commits, pushes, and connects what it declares. Then open Customize, choose Connectors, and connect the Greenhouse and LinkedIn accounts it reads with. Connector credentials are brokered server-side and never enter the machine. Run one sweep by hand before you trust the schedule:

```bash
kortix sessions new --agent candidate-sourcing --prompt "Load the outbound-sourcing skill and run one complete sourcing sweep for the open role."
```

One isolated sandbox per session. Each session has its own isolated machine and branch. A bad run costs one branch and no state. Read every draft, then let the trigger take over at 07:00 UTC.

## Where the stop rules live

Four files hold them, and a change to any of them is a diff you review before it lands. `kortix.yaml` holds the two policy sets. Greenhouse exposes `get_*` and `list_*` reads and holds everything else for a person, and LinkedIn exposes `search_*` and `get_*` reads. With `policy.default_mode: risk`, an unlisted write waits even if a connector update adds a new action. `agents/candidate-sourcing.md` states what the run may do, in the words the model reads first. `skills/outbound-sourcing/SKILL.md` turns the four guardrails into an ordered checklist, including the pipeline read that has to finish before the first draft exists. `docs/dedupe-rules.md` defines the match keys and the skip record, so an empty skip list is treated as a bug in the run. Approval gates you set. Off until you set them. That default is why the policies ship as YAML in the manifest.

## What this is not

No candidate database, no send action, no ATS writes. The repository sources and drafts, and a person sends. `docs/what-it-is-not.md` lists the boundaries in full and names the case where a sourcing product with its own index is the better purchase.

## Where to read more

The [how-to guide](https://candidatesourcingguide.com/how-to-source-candidates) covers the manual process this agent replaces, and is the place to start if you have never built a boolean search for a role. If you are weighing a keyword tool against a model that reads each profile, [AI candidate sourcing](https://candidatesourcingguide.com/ai-candidate-sourcing) covers where the two part ways. [Candidate sourcing vs recruiting](https://candidatesourcingguide.com/candidate-sourcing-vs-recruiting) draws the line this repository stops at: sourcing finds people and drafts the first message, recruiting decides. The whole set, including what a first run should look like, is at [candidatesourcingguide.com](https://candidatesourcingguide.com). If you want to know what you can inspect and change in a setup like this, [open-source candidate sourcing](https://candidatesourcingguide.com/open-source-candidate-sourcing) covers it. The [candidate sourcing tools](https://candidatesourcingguide.com/candidate-sourcing-tools) page compares the products that ship a database, a category this repo sits outside. Start a project at [kortix.com](https://kortix.com).
