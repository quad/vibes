---
name: build-skill
description: Design a new SKILL.md — pick the structure that fits the job, encode only what the model would otherwise get wrong, and refine through real-use retrospectives. Use when the user wants to create or rewrite a skill.
allowed-tools: Read, Edit, Write, Grep, Glob, Bash, Agent, AskUserQuestion
argument-hint: <skill-name-or-purpose>
---

# Build a skill

**Argument:** the name or purpose of the skill the user wants to write. If unclear, ask once before drafting.

## Posture

- **A skill is a loader specification, not a prompt.** What goes into context, when, and at what cost matters more than the prose itself. Architecture beats wording.
- **Encode only what the model would otherwise get wrong.** Things the model already does correctly don't need to be in the skill — specifying them crowds out judgment, doesn't improve compliance, and triggers obedience reinforcement that hides better options. Only three categories earn their keep: environment-specific overrides of model priors (Gotchas), non-obvious mechanics the model can't infer (e.g. specific API shapes or ID distinctions), and deliberate posture overrides (e.g. "do good work, not safe work").
- **Imperative phrasing beats negation.** "Stop and ask if Y" is followed more reliably than "Don't fail to ask if Y." Reserve "never" for genuine inviolable rules.
- **Skills must be self-contained.** Don't depend on other skills. If a pattern is reused across skills, copy it; don't reference an external file. Sharing should be drag-and-drop.
- **The skill embodies its own principles.** A meta-skill that preaches brevity must itself be brief; a skill that says "verify against the code" must be verifiable from its own examples.

## Boundary

- Frontmatter belongs only on the SKILL.md file. Never put it on `references/` siblings — it promotes them to skill-level visibility and confuses the loader.
- Don't include table-stakes rules ("don't be malicious", "be helpful"). They dilute signal and crowd out the real overrides.
- Don't explain things the model already knows. The audience is a model trained on enormous amounts of code and prose. Stop one level above the model's prior, not below it.
- Stay under ~500 lines for the SKILL.md body. Above that, split into a thin spine plus `references/EXAMPLES.md`, `references/COMMANDS.md`, etc., loaded on demand.

## Vocabulary

| Name | Meaning |
|---|---|
| Frontmatter | YAML at the top. Loaded every turn (cost-bearing). Standard fields: `name`, `description`, `allowed-tools`, optionally `argument-hint`. |
| Body | The SKILL.md content below frontmatter. Loaded only on invocation. |
| References | Sibling files (`references/*.md`, `scripts/*`) loaded on demand. Zero context cost until used. |
| **Genre** | The shape pattern that fits the skill's job — see §2. |
| **Section** | Standard parts: Trigger, Posture, Boundary, Vocabulary, Workflow, Gotchas. Use only what earns its place. |

## Workflow

### 1. Understand the user's actual problem

Before drafting anything, find out:

- What task is the skill for? Get a concrete invocation example.
- What does the model currently get wrong without help? Specifically — not "would be helpful to remind it," but "fails or struggles when not told."
- What's environment-specific? Tools the user has, conventions of the codebase, preferences in their memory files.
- When should the model invoke this? What's the trigger?

If you can't answer all four, ask the user. Don't fill in defaults.

If the task is one-off, the answer might be "don't make a skill" — a one-liner instruction at invocation time is cheaper than a skill nobody invokes again.

### 2. Pick the genre

Match the skill's job to a known shape. Most polished skills fit one of these:

- **Recipe** (~30-70 lines): a single procedure with prerequisites and one or two commands. The skill is mostly a connection pattern. Sections: Trigger + Prerequisites + Procedure + Notes.
- **Workflow** (~150-300 lines): multi-step process with branches and state threaded through. Sections: Posture + Boundary + Vocabulary + numbered Workflow + Gotchas.
- **Pattern-applier** (~80-150 lines): reusable approach for a class of problems, applied per-invocation to a different target. Sections: Posture + Workflow.
- **Reference** (variable): dense lookup material. Mostly tables or listings the model needs at hand for a domain.
- **Triage / field-guide**: classifies inputs into action classes. Often a sub-section of a workflow rather than a whole skill.

If the skill spans genres, pick the dominant one and treat the others as sections within it.

The pattern-language master frame fits all of these: Context (when this applies) → Forces (constraints pulling in different directions) → Solution (what to do) → Consequences (what you trade off). Use it as a sanity check.

### 3. Draft the spine

Write only the SKILL.md body. Defer references and scripts until §5.

For each candidate section, ask: "If I removed this, would the model still do the right thing?" Cut anything that survives the question.

- **Posture** earns its place when the model's defaults need overriding (e.g. it would route too much to the user; tell it to exercise judgment).
- **Boundary** earns its place when there are inviolable rules the model would otherwise plausibly cross (e.g. don't push, don't rewrite history, don't post without a confirmation).
- **Vocabulary** earns its place when structured data threads through the workflow and consistent naming matters.
- **Workflow** earns its place when the order matters and getting steps wrong has bad consequences.
- **Gotchas** earns its place for environment-specific overrides of model priors — typically the most maintenance-load-bearing section.

Front-load critical content. Models attend more to the start and end of long contexts than to the middle, so Posture and Boundary should appear before Workflow.

Phrase positively wherever you can. Imperative > negation.

### 4. Ablate against the model's prior

For every line, ask whether a competent model would do this without being told. If yes, cut it. The risk of leaving it in isn't neutrality — obedience reinforcement makes the model follow the spelled-out instruction even when its own judgment would have produced a better result.

This is the hardest step. Authors who care about a domain instinctively over-specify. Resist.

### 5. Split if the body exceeds ~500 lines

Extract material to `references/` siblings (no frontmatter on them — see Boundary). Common patterns: `references/EXAMPLES.md`, `references/COMMANDS.md`, `references/REFERENCE.md`. Reference them by name from the spine; the loader brings them in on demand.

### 6. Run on a real task

The skill is not finished until it has been invoked on a real instance of the work it's meant to do. Models behave differently than reviewers predict. Reality beats simulation.

### 7. Transcript retrospective

After each real invocation, scan the session log for friction. Session JSONLs live in `~/.claude/projects/<project-path>/<session-id>.jsonl` and are usually too large to read directly — use focused subagents.

The pattern that works:

1. Spawn 3-4 `Agent` subagents in parallel. Each gets the same session file path and a different question:
   - "Where did the model get stuck or take an unexpected path?"
   - "Where did the user push back or correct?"
   - "What got produced — is it good work?"
   - "What new failure modes surfaced that weren't covered by the skill?"
2. Tell each subagent the relevant content is at the end of the file and to use `tail` and `Read` with offset (so it doesn't read the whole 4MB JSONL).
3. Synthesize. Separate genuine bugs from philosophical objections you've already overruled.
4. Apply surgical edits to the skill. Don't rewrite.

Iterate until new sessions surface only objections you've already overruled. That's the stop signal.

## Gotchas

- **`AskUserQuestion` availability is unreliable.** It can be listed in `allowed-tools` yet have no schema loaded (verify with `ToolSearch select:AskUserQuestion`). If the new skill uses it, include a plain-text fallback path.
- **`allowed-tools` doesn't restrict.** It pre-approves listed tools (skipping per-use prompts) but doesn't prevent the agent from using others. It's also CLI-only — has no effect in the SDK.
- **The `description` field is loaded every turn.** Keep it under ~30 words and front-load the trigger context.
- **Live change detection is environment-dependent.** Claude Code CLI watches for file changes mid-session; ACP-based clients (Zed) often cache skills at session start. After editing a skill, start a fresh session to test.
- **Common bad-skill patterns to avoid:** all-caps exhortations ("CRITICAL", "NEVER"); chatty meta-preamble before the actual content; restating frontmatter fields inline; mixing table-stakes ("don't be malicious") with non-obvious tradeoffs; magic numbers without justification ("retry 3 times" — why 3?).
- **Skills are calibrated to model behavior.** A stronger model may follow instructions more literally and produce worse output. Re-run §6 on every model upgrade.
