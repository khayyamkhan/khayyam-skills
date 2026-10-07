---
name: state-space-search
description: Use this skill whenever the user needs to figure out the best order to do something, or wants to evaluate multiple possible action sequences against goals, constraints, dependencies, or risks. Trigger this for questions like "what should I do first," "does it matter if I do X before Y," "what's the safest order to roll this out," "which approach has less downstream risk," or any request to compare a handful of possible plans, migrations, rollouts, or workflows. This applies far beyond software — UI/UX flows, product launches, ops planning, film/VFX pipelines, business strategy, curriculum design, legal or compliance workflows, robotics, game design, or personal planning all qualify. Reach for this skill even if the user never says "sequence," "order," or "state space" explicitly — if they describe a goal plus several possible actions/steps with dependencies, constraints, conflicts, or irreversible consequences, this skill applies and should be offered proactively, not just when asked.
---

# State Space Search

A method for evaluating possible orders of operations by simulating how each action changes state, affects constraints, creates downstream consequences, and changes the likelihood of success. Use this instead of just listing tradeoffs when the problem is really about *sequencing* — where doing something first enables or blocks something later.

Don't just compare options side by side. Model the sequences.

## When this applies

This is the right tool whenever a decision has three ingredients together: multiple candidate actions, a goal, and dependencies or constraints that make order matter. If order genuinely doesn't matter (the actions are independent), say so plainly instead of forcing this framework onto a problem that doesn't need it.

Domain doesn't matter — code changes, product rollouts, film/VFX pipelines, business strategy, legal workflows, game mechanics, curriculum design, personal planning, robotics, and agent workflows all fit. Don't assume the domain is software unless the user says so.

## Core mental model

Treat every candidate action as a state transition:

```
Current State → Operation → State Change → Constraint Impact → Downstream Consequence → Fitness Score
```

For each action under consideration, ask:

```
If this happens first, what does it enable?
If this happens first, what does it block?
If this changes the state, does the next operation still work?
If the final result fails, which earlier decision caused the failure?
```

## Step 1: Extract the model

Pull the problem apart into these pieces. If something is missing, make a reasonable assumption and label it clearly as an assumption rather than stalling to ask — only ask the user if the missing piece makes the task genuinely impossible to reason about.

- **Goal**: what outcome are we actually trying to reach?
- **Operations**: the candidate actions/steps. For each one, note what it `requires` (preconditions), what it `creates`, what it `removes`, what it `blocks` afterward, what it `conflicts_with`, its rough `cost`/`value`/`reversibility`, and your `confidence` in these assumptions.
- **Initial state**: known facts, open uncertainties, hard constraints.
- **Success criteria** and **failure conditions**.

Keep this lightweight for small problems (a short list is fine) — don't force a heavy schema onto a 2-step decision.

## Step 2: Generate and simulate sequences

- For 3–5 operations, consider full permutations (3 ops = 6 sequences, 4 ops = 24, 5 ops = 120) — you don't need to write all of them out, but reason through enough of them to be confident you've found the strong candidates.
- For larger sets, prune early: eliminate sequences that violate an obvious precondition before spending time simulating them step by step.
- For each promising sequence, walk it forward: start at the initial state, apply each operation in order, update the state, and check whether the next operation's preconditions still hold. Note any new risks or newly-blocked actions as they appear.

Reject or penalize a sequence when it does any of the following: requires a state that doesn't exist yet, removes something a later step needs, creates a conflict with a future step, causes unnecessary rework, locks in a decision earlier than necessary, delays a critical dependency, increases ambiguity or fragility, or makes later steps more expensive than they need to be.

## Step 3: Trace failures backward

When a sequence fails, don't stop at the immediate cause — trace it back to the decision that's actually responsible:

```
What did the failed step require?
Which earlier step should have created that requirement, but didn't?
Which earlier step removed, blocked, or destabilized it instead?
Which decision deserves the blame?
```

This is the most valuable part of the analysis for the user — it's the difference between "step 4 failed" and "step 4 failed because step 1 threw away something step 4 needed, and that wasn't obvious until now."

## Step 4: Score and recommend

Compare the surviving sequences across whatever dimensions matter for this problem — typically some mix of goal fit, constraint satisfaction, risk, complexity, reversibility, efficiency, downstream flexibility, and confidence in the assumptions. You don't need a rigid 0–10 rubric for every problem; scale the rigor to the stakes. A high-stakes irreversible decision earns a fuller breakdown than a quick personal-planning question.

Then recommend one sequence and be precise about why. Don't say "this seems better" — say what it does:

> This sequence wins because it creates the required dependency before it's consumed, avoids locking the system in too early, and preserves the most reversibility if step 3 goes wrong.

Always name the runner-up and what would have to change for it to win instead — that's often as useful to the user as the recommendation itself.

## Output shape

Scale the output to the size of the problem. For a substantial, multi-step decision, cover:

1. The goal, in one line.
2. Assumptions made (only the necessary ones).
3. The operations, normalized.
4. The candidate sequences actually worth discussing (not every permutation, unless asked).
5. What fails, and why, for the weak ones.
6. The recommended sequence and the reasoning tied to state changes and consequences.
7. The biggest risk in the recommended path, and what would flip the recommendation.

For a smaller or lower-stakes question, compress this into a few sentences: best order, why, the one thing to watch out for. Don't pad a simple answer with the full structure just to look thorough.

## Guiding principles

- Prefer traceable reasoning over intuition — every recommendation should be traceable through decision → state change → consequence → score.
- Separate hard constraints (must hold) from soft preferences (nice to have) explicitly; don't let them blur together.
- Separate assumptions you made from facts the user gave you.
- When confidence is low, say what additional information would sharpen the ranking, rather than presenting a guess as settled.
- When the search space is small, actually enumerate it. When it's large, prune early and say what you pruned and why.
