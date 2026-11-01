# How a run works in the open-source Kortix candidate sourcing agent

Kortix is open source. One run of this agent is a fixed sequence: a cron fires, a session starts on its own machine, the pipeline is read, the search runs, the drafts are written, and the batch lands in a review queue with nothing sent. The example numbers below come from the run Kortix published at [kortix.com/use-cases/candidate-sourcing](https://kortix.com/use-cases/candidate-sourcing).

## 07:00, the trigger fires

The manifest in this repository carries one trigger. It starts one session a day with a fixed prompt, and nothing else runs the agent.

```yaml
triggers:
  - slug: daily-sourcing-sweep
    name: Daily candidate-sourcing sweep
    "on":
      cron: "0 0 7 * * *"
      timezone: UTC
    run:
      agent: candidate-sourcing
      prompt: Load the outbound-sourcing skill and run one complete sourcing sweep for the open role.
```

A schedule starts a session and nothing else. No queue worker holds the role, no table tracks progress between runs, and no state survives except what the session writes out. Change the `cron` line and the run moves; set `enabled: false` and it stops.

## The session starts

One isolated sandbox per session. Each session has its own isolated machine and branch. That is the property that makes a daily run safe to leave alone: a run that goes wrong can read the wrong search results and still cannot reach the previous run's files or another session's.

## The pipeline read comes first

The first connector call opens the role's Greenhouse pipeline. That read produces the set of people the company already knows for this role, and the set is what the rest of the run is measured against. In the published run, the check removed thirteen people from the list before a single email existed.

A pipeline read that returns nothing is treated as a failure. See `dedupe-rules.md` for the match keys, and for the reason an empty skip list is the loudest bug a run can report.

## The search

LinkedIn returns profiles for the role's search. Every profile is read before it counts: the current role, the tenure, the work the person actually did. In the published run, thirty-one profiles passed that bar. Reading is the slow part and the part a reviewer should sample, because a profile that matches on headline alone is the most common way a batch fills with people who will not reply.

## The drafts

Eighteen drafts, one per person the pipeline did not already hold, each naming the candidate's current role and the reason they matched. Every draft passed the deletion test in the skill: remove the name and the current role, and the paragraph stops working. This is also where a run can fail quietly. A batch of eighteen drafts that all open the same way is a run that searched correctly and wrote badly. The reviewer catches it by reading three drafts in a row.

## The review queue

The run ends with one batch in the review inbox. Each item is one candidate, and the batch is the last thing the run writes.

```
Role: Senior backend engineer
Profiles reviewed: 31
Dropped as already in pipeline: 13
Drafts written: 18
Pipeline read at: 2026-11-01T07:02:14Z
```

Each item in the batch carries the name, the current role and company, the profile URL, one sentence on the match, the dedupe result, and the draft. A recruiter opens the batch, edits what needs editing, sends what they choose, and decides whether the person goes into Greenhouse.

Nothing in the run touches the ATS and nothing leaves the building. That boundary is the whole design, and it is why the batch is the last artifact of a run.

## What the next run inherits

Nothing from the previous session. The next run reads the pipeline again, which is where yesterday's sends now live, and that read is what keeps a person who replied yesterday from getting a second email today.
