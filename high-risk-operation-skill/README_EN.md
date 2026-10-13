# High-Risk Operation · Evidence Discipline for Irreversible Actions

**A lightweight, zero-dependency Skill + prompt pack: make an AI state its evidence and its limits *before* deleting, uninstalling, or cleaning anything that cannot be undone.**

[中文说明](README.md) · [Skill (Chinese)](skills/high-risk-operation/SKILL.md)

---

## What this is

People ask AI to clean up disks, uninstall software, remove duplicate files. It usually answers promptly:

> "This directory is a redundant copy — safe to delete."
> "This executable has no digital signature, origin unknown, recommend removal."
> "Confirmed identical to the source file."

So you delete it. **Then you find out.**

The problem isn't that the AI lies. It's that **when it says these things, it doesn't know that it hasn't verified them.**

This is called **treating an assumption as a measurement.**

This Skill does exactly one thing: **it forces the nature of each conclusion to be labeled.**

| Tag | Meaning | Can it justify a deletion? |
|---|---|---|
| **[Verified]** | Obtained by running a command / reading a file in this session | ✅ Only this can |
| **[Inferred]** | Derived from verified data | ⚠️ Must state the derivation |
| **[Assumed]** | An unverified guess | ❌ **Never sufficient** |

---

## Where it came from

Not a rule list invented from scratch — it was reverse-engineered from **a chain of real incidents.**

Four documented failure patterns:

| Symptom | Actual consequence |
|---|---|
| Read only a **version-info field** of an executable, then declared "no digital signature" | It had a valid signature from an authoritative CA — the judgment was exactly backwards |
| Copied **terminal output from memory** into a table, then claimed "cross-checked against program output" | The table did not match the real output, and only item-by-item checking would reveal it |
| Saw two directories that "**looked** identical" and declared one a redundant copy | **Not a single file matched in size** — they were not copies at all |
| Looked at **blurry text** in a rendered thumbnail and announced "figures verified" | Never checked the original pixels; the conclusion had no basis |

The common thread: **conclusions came from "I remember", "I infer", or "it looks like" — not from "I just read a file."**

Hence this Skill.

---

## Quick start

### Option 1: Copy the prompt (works with any AI)

```
You are helping me perform irreversible operations: deletion, uninstallation,
cleanup, registry edits. Follow Evidence Discipline throughout:

1. Before acting, state each target's absolute path, file count, and size.
2. Label every conclusion: [Verified] obtained by running a command / reading a
   file this session; [Inferred] derived from verified data (state the basis);
   [Assumed] unverified guess. ONLY [Verified] can justify a deletion.
3. To claim something is "safe to clean", prove it via reference/dependency
   relationships -- never by file or directory names.
4. State whether the action is reversible, whether a backup exists, and whether
   it can be turned into a move instead of a delete.
5. State your own limits: permissions, whether targets are locked, whether a
   reboot is required.
6. Act one target at a time; report partial failures honestly; do not touch
   anything outside the requested scope.
7. After acting, verify (rescan / compare hashes / check for leftovers).
   Never assume success.
8. Distinguish "functionality removed" from "residual files left behind".
9. End with a verification block whose "Unverified" field must not be a vague "none".

Now check whether your next action satisfies all of the above. If not, fix that first.
```

### Option 2: Install the Skill

Copy the whole `skills/high-risk-operation` directory into your skills folder:

```bash
# Claude Code
cp -r skills/high-risk-operation ~/.claude/skills/

# Other agent tools: place it in their skills directory
```

Then just say "help me clean up my disk" or "can these files be deleted" — it will follow the evidence discipline automatically.

---

## The six-step flow

| Step | Do what | Key requirement |
|---|---|---|
| 1 | State what will be touched | Absolute paths + file counts + sizes |
| 2 | Supply evidence, labeled | [Verified] / [Inferred] / [Assumed] |
| 2b | Use reference relationships, not "looks like" | Names are not evidence; references are |
| 3 | Judge reversibility | Can it be a move? Is there a backup? |
| 4 | Declare your own limits | Permissions, locks, reboot needed |
| 5 | Act | Per-target reporting; nothing out of scope |
| 6 | Verify | Rescan, and check what *shouldn't* have changed |

---

## Verification block template

Required after every irreversible operation:

```
── Operation verification ──
Targets      : <absolute paths>
Before       : <file count / size>
Action       : <exact command or operation>
Result       : N succeeded / M failed
After        : <file count / size from rescan>
Collateral   : <are the artifacts that shouldn't have changed intact?>
Leftovers    : <list with impact, if any>
Unverified   : <explicitly list what was NOT verified>
```

**The "Unverified" field may not be a vague "none".**

---

## Scope and limits (important)

This Skill is a **behavioral constraint**, not a **safety checker**. It **cannot**:

- Replace backups
- Decide whether a file matters **to you** (only you know that)
- Stop a model that is already certain it is right

All it can do is **make the nature of each conclusion and the limits of each operation visible, so the user has a chance to stop it before anything happens.**

**If you remember only one line:**
> Before acting, any conclusion without [Verified] backing is not a conclusion.

---

## When to use

- Disk cleanup, removing redundant files
- Uninstalling software, cleaning the registry
- Judging "duplicate" or "useless" files
- Anything that cannot be undone

## When not to use

- Read-only analysis (no changes involved)
- Code edits under version control with easy rollback

---

## Repository layout

```
high-risk-operation-skill/
├── skills/
│   └── high-risk-operation/
│       └── SKILL.md          # Skill (Chinese, primary)
├── README.md                 # 中文说明
├── README_EN.md              # This file
└── LICENSE                   # MIT
```

---

## License

MIT. PRs welcome.

---

## One aside

The origin of this Skill is a little ironic:

It is **an AI writing down its own mistakes in order to constrain itself.**

And whether these rules actually work depends on **whether the user is willing to ask one more question when the AI says "confirmed."**

Tools cannot cure overconfidence — they can only make it visible.
