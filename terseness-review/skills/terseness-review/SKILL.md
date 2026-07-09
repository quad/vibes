---
name: terseness-review
description: >-
  Tighten the prose in your current VCS changes — comments, docstrings, commit
  messages, touched Markdown — against a terse, why-focused standard, then apply
  the trims directly. Use on /terseness-review, or when asked to trim / prune /
  clean up comments or commit messages, to check whether prose restates the code
  or is too verbose, to strip stream-of-consciousness rationale before a PR, or
  to catch relative dates and internals-leakage in changed prose — even when the
  word "terseness" is absent.
---

# Terseness Review

Judge every line of prose *you added or changed* against one reader: a busy
engineer with the diff, the code, the identifiers, and the UI already open.

**The test, per sentence:** would that reader gain something here they can't get
from what's already in front of them? No → cut. Yes → keep only that, said once.

Length is downstream. Cut *redundancy* and *internals-leakage*, not line count —
a four-line docstring is right when four distinct things earn their place. And
touch only prose the change already touches: trimming is not license for a
drive-by that enlarges the diff.

## Workflow

1. **Gather** the changed prose — see [Gathering](#gathering).
2. **Extract targets:** natural-language content on added/modified (`+`) lines —
   comments, docstrings, help-text string literals, Markdown prose — plus every
   commit message on the branch. Ignore pure code.
3. **Judge** each against [The standard](#the-standard): `cut`, `trim`, or
   `keep`. Most diffs yield few findings; don't manufacture them.
4. **Apply** immediately — no confirmation gate; VCS makes it recoverable. See
   [Applying](#applying).
5. **Report** what changed — see [Report](#report).

## The standard

The failure is almost always one principle misapplied and one honest signal
mistaken for noise. Both discriminators below matter more than "be brief."

### The WHY trap — the mistake to watch for

"Explain WHY" gets misread as "narrate the implementation's rationale." Only one
flavor of WHY belongs in code:

- ✅ **Caller-visible** — the contract seen from outside. *"Throws if the batch
  is already posted."* *"Pass a stable id so telemetry can query by it."*
- ❌ **Implementation-rationale** — design history. *"We chose this ordering
  because…"* *"Could be reordered but then…"* *"Cast because the generic is
  wider."* Read once, never again → belongs in the commit message, not the code.

Discriminator: name the *external observation* the reader gains. Can't? Cut it.
Tests catch reorderings; commit messages preserve history; the code is the choice.

### Failure modes — flag these

- **Restates-WHAT** — narrates the next line or the diff. `// increment` above
  `count++`; a commit body bullet-listing the files it added.
- **Implementation-rationale** — WHY-trap design history in code.
- **Internals-leakage** — exposes private structure to a behavior-only reader;
  commit subjects describing wiring rather than the user/operator-visible change.
- **Redundant-idiom** — restates an idiom. `// best-effort` beside `?.`;
  `// narrow the type` beside `as U`.
- **Step-narration** — prose mirroring mechanical steps, common in tests.
  *"Close, then re-create on the same object"* above two lines doing that;
  section labels like *"Histograms are skipped."* when the assertion says so.
- **Stream-of-consciousness** — paragraph-long shape rationale. The code is the
  shape; future-modifier WHY goes in the commit message.
- **Relative-date** — "today"/"last week"/"recently" a future reader can't
  resolve. → ISO date (past), absolute target or urgency interval like "within a
  week" (future), or event description "after ~4 weeks of data" (scheduled). If
  the real date can't be recovered — e.g. when relocating rationale into a commit
  message — describe the event without the time word ("a prior gateway
  incident"). Never keep "last week", never invent a date.
- **Speculative-guard** — a comment justifying a defensive check with no namable
  trigger. If none can be named, the check itself is suspect — flag and say so.
- **Drive-by** — reformatting/renames/cleanup unrelated to the change's purpose.

### Earns its place — leave these alone

Flagging these is the worse error; they carry signal the reader can't reconstruct.

- Operator-facing names — metric/span names, `pool.instance.id`, dashboard query
  targets.
- Call-site contract corners — *"double-retire is a no-op"* changes retry logic.
- Constraints the code can't express — *"must run inside a Temporal activity."*
- Non-obvious test math — *"wait_ms = end-of-acquire (1500) − start (1000)."*
- FAQ-preempts whose answer isn't googleable — a vendor quirk, a library's
  non-obvious side effect.

Commit subjects: prefer **subject-as-WHY, no body**, unless the body carries
non-derivable context (design history, a rejected alternative, a migration
caveat). A body that restates the diff → cut to the subject.

## Gathering

Detect the VCS once. Prefer jj where present (`jj root` succeeds).

**jj:**
- Diff: `jj diff --git -r 'trunk()..@'` (or `jj diff --git` for the working copy
  alone if the user scopes it that way). Never read the non-`--git` format — its
  line-number pairing misreads easily.
- Messages: `jj log -r 'trunk()..@' --no-graph -T 'change_id.short() ++ "\n" ++ description ++ "\n---\n"'`

**git** (fallback): `git diff origin/main...HEAD` + `git diff HEAD` for prose;
`git log origin/main..HEAD --format='%H%n%B%n---'` for messages.

If `trunk()` / `origin/main` won't resolve, fall back to the working copy and say so.

## Applying

Land each fix where the prose lives, so history stays clean.

**jj:**
- **Code/doc prose:** Edit the working copy, then `jj absorb`. Absorb routes each
  hunk back into the branch commit that last touched those lines; unmatched hunks
  stay on `@` — exactly the intent, no manual targeting. Confirm with
  `jj log -r 'trunk()..@'`.
- **Commit messages:** not file hunks, so absorb won't move them. Rewrite with
  `jj describe -r <change_id> -m "<new message>"`.

**git:** apply edits, then one commit `style: tighten changed prose`. Rewrite
branch *messages* only if asked — mid-branch history rewriting is riskier here.

Honor repo VCS conventions (CLAUDE.md / CLAUDE.local.md): don't push, don't
change the working revision.

## Report

One-line tally, findings by location in diff order, then what landed. The report
obeys the standard too — keep it scannable.

```
Terseness review — <N> findings across <M> locations

<file:line | change_id>
  ✂ cut   "<prose>"          → <mode>: <one clause why>
  ✎ trim  "<orig>" → "<new>"  (<mode>)
  ✓ kept  "<prose>" — <why>   [only when a keep was non-obvious]

Applied: <edits made · hunks absorbed into which commits · messages rewritten ·
          what stayed on @>
```

No findings is the expected outcome for already-tight prose — say so in one line,
not as a failure.
