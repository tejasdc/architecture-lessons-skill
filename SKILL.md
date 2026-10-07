---
name: architecture-lessons
description: Use BEFORE adding a value, list, enum, option set, limit or default that more than one file, package, service or repository will need — a model list, a dropdown's options, a status vocabulary, a timeout, a feature's allowed values — and before changing one that already exists in more than one place. Triggers on "I thought we agreed to change this everywhere", "why is it still showing the old one", "we updated it in one place and it didn't flow", "we keep missing this", "make this architecturally impossible", "one source of truth", "single source of truth", "these two lists disagree", "the picker is stale", "the dropdown doesn't have the new one", "hard-coded in two places", "add it to the enum", "where else is this used", "is there a list of everywhere this is referenced". Covers: which piece of knowledge gets one home; the three enforcements ranked by strength (derive so a gap cannot compile, publish so consumers read instead of copying, refuse the second copy with a check); why a derived guard you have not watched fail is not a guard; why a fallback list is the second copy in disguise; and how to inventory the duplicates you already have. Do NOT use for duplicated *code* that merely looks alike — that is not what DRY means and deduplicating it usually makes things worse.
---

# One home per piece of knowledge

This skill owns one architectural rule: duplicated knowledge. For the method of designing or
reviewing a whole system or feature (requirement in the user's words, operating profile, lifetimes
and owners, the invariant, exchanges that end explicitly, rollout that reaches running code, the
comparative decision case, checks before building) load `architecture-foundations`, which routes
here for this rule. On 2026-10-07 we used this skill to judge the journal-versus-ledger split in the
Concierge agent-work design; with Astra we found the rule applies as written (the ledger owns
obligations and cursors, the journal owns observations, and the journal never independently settles
an obligation), and that the rest of the review needed a broader skill, which is why that one exists.

**DRY is about knowledge, not about text.** Hunt and Thomas wrote it as *"every piece of
knowledge must have a single, unambiguous, authoritative representation within a system"*.
Two functions that happen to look alike are not a violation. Two places that both have to be
*told* when something changes are — and that is the failure this skill exists to stop.

The test is not "does this look like something else". It is: **when this fact changes, how
many places must be edited, and what happens if one is missed?** If the answer is "the wrong
thing quietly shows up somewhere", it needs one home now.

## The three enforcements, strongest first

Reach for the strongest one you can afford. A rule nobody *can* break beats one everybody
must remember, and "we'll be careful" has already failed every time this skill has been used.

**1. Derive it, so a gap cannot compile.** Best. Make the second place a *consequence* of the
first rather than a copy of it: a map keyed by a literal union, an exhaustive switch, a type
generated from a schema. A missing entry becomes a build error at the moment it is created.

> **A derived guard is only real once you have watched it fail.** Write the broken version
> on purpose, build it, and see the error. The first attempt at exactly this, on 2026-09-23,
> derived a union through a field typed `string`; the union widened to `string`,
> `Record<string, string>` accepted anything, and a deliberately broken build passed clean.
> The guard was decorative and would have been shipped as a guarantee.

**2. Publish it, so consumers read instead of copying.** Across a process or repository
boundary you cannot share a type, so serve the value and have clients render what arrives.
The owner gets an endpoint; the client keeps nothing.

> **A fallback list is the second copy in disguise.** When the published list cannot be
> reached, say so and offer the safe default. Do not "temporarily" hold a copy for that case
> — it will be the copy that goes stale, and it will look like it is working.

**3. Refuse the second copy with a check.** A lint or architecture test that rejects the
literal anywhere but its one home. Name known exceptions *in the rule, with their reason*, so
they stay visible and a genuinely new copy is still caught. An exception list that is empty
because you widened the pattern is not enforcement.

## When you find one, inventory the rest

The copy he noticed is rarely the only one. Sweep both directions — same repo and across
repos — for the same shape: option lists mirroring a backend union, status vocabularies,
limits written twice. Report each as *what / where each copy is / do they agree today*, and
fix the ones that have already drifted. For copies you are not fixing now, **write them down
somewhere durable with a reason.** A known duplicate is survivable; an unknown one is what
reaches his screen.

## The incident this came from

On 2026-09-23, Concierge adopted Claude Opus 5.5. thnkr.ing's new-session picker went on
offering "Claude Opus 5" and "Claude Opus 5 1M", because the same list was also written out
in the web app twice — once as the picker's options, once as the display names.

> "Why do I not see Opus 5.5 in my list of provider models here? I thought we agreed to
> change this everywhere. Looks like we don't have a comprehensive list of everywhere the
> model is being used and tied. I think we need to make this architecturally impossible. Our
> architecture should be such that there's one source of information for these things, for
> agents and for models, one configuration or one setting, and that should flow into the
> different places that use it. Otherwise we're going to keep missing this. A lot of times an
> update is made in some places and doesn't flow to other places." — Tejas

Sweeping for others found three more the same drift had already reached, none of them the
picker he was looking at: a local runtime that **defaulted to a retired model**, so unlike the
display bugs it changed which model actually ran; a provider dropdown listing ten of fifteen
options, already six behind; and an event-kind list missing an entry, so one kind of update
never refreshed a cached view. Two of the three were invisible from any screen.

**The lesson is not "remember to update the other place."** It is that a value with more than
one home will drift, the drift will be invisible until someone notices the wrong thing on a
screen, and the copies you find by looking are usually worse than the one that was reported.

See `references/inventory-method.md` for how to run the sweep.
