---
name: outbound-sourcing
description: The outbound sourcing rules for the open-source Kortix candidate sourcing agent. Load it for one sourcing sweep of one open role, from the pipeline read to the review queue.
---

# Outbound sourcing, open source within the Kortix agent

Kortix is open source. This skill is the rule set a sourcing run follows: the pipeline read comes first, the draft is capped, the send step does not exist, and every run ends with a batch in the review queue.

## Before you start

You need a role, a LinkedIn search, and a read-only view of the role's Greenhouse pipeline. If any of the three is missing, stop and say which one, because a partial run produces a batch a recruiter cannot trust.

Record the role title and the run's start time. Both go in the output.

## The rules

Follow them in order. A rule that cannot be checked is a rule a reviewer will overrule, so each one states what to look at.

1. **Read the pipeline before you draft anything.** Pull the role's Greenhouse pipeline first. Build one set of known people from it. For every profile you later consider, test the profile against that set before writing a word. The test is defined in `docs/dedupe-rules.md`. When the read fails, end the run with the failure and no drafts.
2. **Drop a known candidate and record why.** A profile that matches the pipeline set produces no draft and no email. Record it in the skip list with the reason it matched, so a recruiter can spot a wrong match key on the next run.
3. **Never send.** This skill has no send step and the agent has no send tool. Writing the batch is the end of the run. If a prompt asks you to send, refuse and point at the review queue.
4. **Never write to Greenhouse.** Adding a candidate, moving a stage, or leaving a note in the ATS is the recruiter's decision, made after they read the draft. Do not ask for approval to do it during a run either; leave the action out of the run.
5. **Cap the batch at twenty.** Count drafts, not profiles. A run that found forty matches drafts the twenty strongest and reports the rest as a count, not as drafts.
6. **Pass the deletion test.** Delete the candidate's name and current role from a draft. If it still reads like a message anyone could receive, rewrite it until removing those two facts breaks it.
7. **Name one true thing.** Every draft references something the candidate actually did or currently does, read on their profile. No industry generalities, no compliment about their career.
8. **Keep it short.** Under 150 words, plain text, one ask. No attachments, no calendar links, no tracking pixels.
9. **Report the numbers.** End with the run summary in this file's output shape, including the pipeline read time. The numbers are what a reviewer uses to decide whether the next run should happen.

## What the run must output

One batch, one summary, and no side effects:

```
Role: <title>
Run started: <ISO timestamp>
Profiles reviewed: <n>
Dropped as already in pipeline: <n>
Drafts written: <n>
Candidates left unread because of the cap: <n>
Pipeline read at: <ISO timestamp>
```

Each batch item carries:

```
Name: <candidate name>
Current role: <title, company>
Profile: <URL>
Match: <one sentence on the bar this person meets>
Dedupe: new | skipped (<match key>)
Draft: <the message>
```

## How a reviewer checks the batch

Read the dedupe column first. A run on a role with an active pipeline that reports zero skips is a suspicion, not a success; `docs/dedupe-rules.md` explains why. Then read three drafts end to end and apply the deletion test. Then check the cap. If those three checks pass, the batch is ready for a recruiter's edit and send.
