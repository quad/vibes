# vibes

Mad skillz.

## Install

```
/plugin marketplace add quad/vibes
```

Then install whichever skills you want:

## Skills

### build-skill

Draft or rewrite a `SKILL.md`. Takes the skill name or its purpose.

```
/plugin install build-skill@vibes
```

### fix-pr

Work through unresolved PR review comments: triage, fix with tests, reply on confirmation. Takes a PR URL or, in a checkout, a bare number.

```
/plugin install fix-pr@vibes
```

### multi-perspective-review

Review a non-trivial artifact by spawning several subagents in parallel, each with a deliberately different lens, then synthesizing. Takes a target — file path, section, or topic.

```
/plugin install multi-perspective-review@vibes
```

### terseness-review

Tighten the prose in your current changes — comments, docstrings, commit messages, touched Markdown — against a terse, why-focused standard, then apply the trims to version control (jj `absorb`/`describe`, or git). Reviews only what you changed; no drive-by.

```
/plugin install terseness-review@vibes
```
