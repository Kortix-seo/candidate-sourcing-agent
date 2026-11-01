# What the open-source Kortix candidate sourcing agent is not

Kortix is open source. This repository is deliberately a narrow piece of software: it searches, it dedupes, it drafts, and it stops. The boundaries below are as much a part of the design as the search step, and a reader should decide whether they fit before running anything.

## It is not a candidate database

The agent searches LinkedIn through a connector. It has no index of its own, no profile cache that survives a run, and no list you can query when the search platform changes its terms or its access. If your sourcing depends on searching a large private index you own, this repo has nothing to offer on that front.

The run also has no enrichment, no contact finding, and no email verification step. A profile with no reachable address is a profile the recruiter deals with, one at a time.

## It has no send action

The run writes drafts and ends. There is no email tool in the agent's grant, no step in the skill that sends, and no scheduled follow-up. A person opens the batch, edits, and sends from their own inbox.

This is the point of the repository. A sourcing agent that can contact real people has to earn trust with a rule someone can read, and the readable version of that rule is a batch that waits for a person.

## It does not write to the applicant tracking system

Adding a candidate, moving a stage, and leaving a note are recruiter decisions made after reading a draft. The connector policy holds writes for a person, and the run does not ask for that approval. The pipeline is read-only for the agent in both the policy and the prompt.

That boundary has a cost. A recruiter who receives eighteen drafts also has eighteen records to create by hand if they choose to add them. The repository does not smooth that step away, because smoothing it is the same as handing the agent write access.

## It does not schedule around your process

There is one cron line, one role per run, and one batch per run. No round-robin across roles, no per-hiring-manager routing, no capacity model that pauses a role when a recruiter is on holiday. The trigger starts at 07:00 UTC and can be edited to another hour; anything more elaborate is a change you make in the manifest.

## When a sourcing product is the better buy

A product that ships its own candidate index, sequences, and an inbox with deliverability tooling solves a different problem, and it solves it for a team that will not run a repository. Four cases point that way:

- You need a searchable database of people across many sources.
- Recruiters on the team will not read a manifest or review a change request.
- You need multi-step sequences with automatic follow-ups and reply detection.
- Procurement needs a vendor contract, a security review, and a named support contact.

None of those are failures of this repository. They are a different shape of purchase.

## What the boundaries are worth

Approval gates you set. Off until you set them. The manifest in this repository sets them, and every one of them is a line a reviewer can read and change before the next run. A team that wants the opposite, fewer gates and a freer agent, can delete those lines. The repository's position is that the deletion should be visible.
