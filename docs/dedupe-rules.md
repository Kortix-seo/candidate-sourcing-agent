# Dedupe rules in the open-source Kortix candidate sourcing agent

Kortix is open source. The dedupe step is the part of this repository to read before you change anything else. It decides who the agent may not write to, and the run is only as good as that decision.

## What the run reads from Greenhouse

The pipeline read happens through the Greenhouse connector in `kortix.yaml`, whose policy set allows the read actions and holds every write for a person. Connector credentials are brokered server-side and never enter the machine.

Four kinds of record decide a skip, and each one is a read from the role's pipeline.

| Record | What it tells the run | What a hit means |
| --- | --- | --- |
| Application for this role | The person applied or was added, in any stage | Skip, at any stage |
| Candidate contact fields | The email the company already holds | Skip when it matches the found profile |
| Stage and status history | Applied, screening, rejected, archived | Skip, including a rejection |
| Outreach activity on the record | Someone has already written to this person | Skip within the lookback window |

The connector's own action names decide how these are called. The skill reads the pipeline with the read actions the policy set allows. It never guesses at an action that is not in the catalog.

## The three match keys

A found profile is compared with the pipeline set on three keys, strongest first. Each key is normalized before the comparison, so one person does not become two.

1. **Email.** Normalized: lowercased, plus-addressing removed, and any alias domain the pipeline marks as an alias folded to the same value. An exact email match is enough to skip, with no second opinion needed.
2. **Profile URL.** Normalized: lowercased host, trailing slash dropped, query string dropped, and the profile path kept as it is. LinkedIn URLs collected by different tools often carry tracking parameters, and those must not turn one person into two.
3. **Name and company.** Matched only when both parts are present. This key is weak on its own, so it never skips on its own. A name-and-company hit with no email or URL match is flagged for a person and held out of the batch.

A single strong key skips. Two keys that disagree, such as a matching name with a different email, hold the candidate out of the batch. The pair goes in the report. That is the one case where the run deliberately stops short of a decision.

## What a skipped candidate looks like

A skip is written down, never dropped silently. Each one is a record in the run output:

```
Name: Dana Whitfield
Matched on: email
Matched value: dana.whitfield@example.test
Source: application for this role, stage screening
Action: skipped, no draft
```

A reviewer can read that block and check the match in under a minute. A skip with no matched value or no source is a bug in the run, not a record.

## Why an empty skip list is a failure signal

A role with an open pipeline almost always has overlap with a search for the same role. People apply, recruiters add people by hand, and earlier rounds leave records behind. The published run dropped thirteen of thirty-one profiles.

Zero skips means one of four things happened, and all four need a person. Check them in this order.

- The pipeline read returned nothing, because the role filter did not match the role's actual record.
- The read failed and the run carried on without it.
- The match keys did not normalize, so the same person came back as two different people.
- The role is genuinely new, with an empty pipeline. That is the only innocent case, and the run should say so in its notes.

Treat a zero-skip run on an established role as a failed dedupe until someone reads the pipeline by hand. The cost of a wrong skip is one missed candidate. The cost of a missing dedupe is a cold email to a person the company already interviewed. That is the failure this repository exists to prevent.

## Where the rules are enforced

Three layers, and each one can stop a bad draft. Read them in this order.

- `kortix.yaml` allows the read actions and holds writes, so the pipeline cannot change during a run.
- `skills/outbound-sourcing/SKILL.md` orders the pipeline read before the first draft and makes the skip record part of the output.
- `docs/dedupe-rules.md`, this file, defines the keys, so a change to matching is a diff a reviewer can read.

Change a key here and the next run uses it. That is the reason the rule lives in a file instead of in a settings form.
