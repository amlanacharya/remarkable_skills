---
name: egm-implement
description: Take an approved concepts brief or RFC (from egm-brainstorm / egm-draft) through spec, GitHub milestone, tracer-bullet tickets, and implementation on one integration branch. Use after the design is settled, when asked to "turn this into tickets and build it", "spec to tickets to code", or to run the implementation phase of the egm family.
---

# EGM Implement: Brief to Spec to Tickets to Code

Implementation phase of the egm skills. Input is a settled design: `concepts_brief.md`, an RFC under `docs/designs/`, or the current conversation. Output is one integration branch with every ticket resolved, under one GitHub milestone.

Adapted from three Matt Pocock skills (`to-spec`, `to-tickets`, `implement-spec`), combined into one flow with a milestone and explicit approval gates.

## Rules that override everything below

1. **Repo rules win.** Read `AGENTS.md` / `CLAUDE.md` first. If they limit commits, pushes, UI changes, staged approvals or testing, follow them over this skill.
2. **Do only what the user asked.** Each outward-facing step (create milestone, publish issues, push, open PR) needs the user's approval at its gate below. Approval for one gate does not cover the next.
3. **Do not interview.** The design is already settled. Synthesize from the brief and conversation. Ask only at the gates.
4. **Plain English.** No filler adjectives. No file paths or code snippets in specs or tickets (they go stale), except a decision-bearing type, schema or state machine.
5. **Report faithfully.** Say what was not run or not verified.

## Phase 1: Spec (gate 1)

1. Read the brief, then explore the repo enough to use its domain vocabulary and respect its ADRs.
2. Pick the **test seams**: the highest existing seam possible. The ideal is one. Propose new seams only when none fits.
3. Write the spec to `.scratch/<feature-slug>/spec.md` with these sections: Problem Statement, Solution, User Stories (long, numbered, "As an <actor>, I want <feature>, so that <benefit>"), Implementation Decisions (modules, interfaces, schema, API contracts), Testing Decisions (behavior only, prior art), Out of Scope, Further Notes.
4. **Gate 1:** show the path, summarize the seams, and wait for approval. Do not publish anything yet.

## Phase 2: Milestone and tickets (gates 2 and 3)

1. Draft **tracer-bullet tickets**: each is a narrow vertical slice through every layer (schema, API, UI, tests), demoable or verifiable alone, and small enough for one fresh context. Do prefactoring first.
   - A **wide mechanical refactor** is the exception: expand (add new beside old), migrate callers in batches, then contract (delete old).
   - Give every ticket its **blocked-by** list. Tickets with none form the starting frontier.
2. Write the draft to `.scratch/<feature-slug>/tickets.md`: numbered, with title, blocked by, what it delivers, acceptance criteria.
3. **Gate 2:** show the numbered breakdown. Ask if the granularity and blocking edges are right. Iterate until approved.
4. **Gate 3:** state exactly what will be created on GitHub (milestone name, spec issue, N ticket issues, labels) and wait for approval. Then:
   - Create the milestone: `gh api repos/{owner}/{repo}/milestones -f title="<feature>"`.
   - Publish the spec as an issue in that milestone.
   - Publish tickets in dependency order (blockers first), each in the milestone, referencing the spec issue as parent and its blockers by issue number. Use native blocking/sub-issue links where `gh` supports them. Apply the `ready-for-agent` label if the repo has it.
   - Do not close or edit any existing issue.

## Phase 3: Implement (gate 4)

**Gate 4:** confirm the integration branch name and whether pushing and a draft PR are allowed. Then:

1. Treat tickets as a **task graph**. The **frontier** is every open ticket whose blockers are done.
2. Create the integration branch from the agreed base. Never reset, clean, force-push or delete branches.
3. For each frontier ticket, start a **fresh implementer subagent** in its own worktree and branch, in the background where possible. Pass only pointers: spec path, ticket number, prior commits. Each implementer:
   - confirms its worktree is based on the integration branch tip;
   - builds the ticket test-first (invoke the `tdd` skill if available);
   - merges the integration tip into its branch before reporting done.
4. When an implementer finishes, a **merger subagent** merges it into the integration branch and re-runs the tests. If this opens new frontier tickets, start them.
5. When all tickets are done, run `code-review` on the integration branch and fix findings in one fresh implementer.
6. If a PR was approved, open it as a draft after the first merge, link the milestone, and mark it ready only at the end. Otherwise report the branch.
7. Remove the implementer worktrees.

## Final report

Tickets completed, tests run and results, anything unverified (UI in particular), open risks, and the milestone and branch links.
