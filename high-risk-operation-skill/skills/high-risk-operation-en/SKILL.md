---
name: high-risk-operation
description: >
  Evidence discipline before irreversible operations. Use when deleting or moving bulk files,
  uninstalling software, cleaning disks, editing the registry or system services, overwriting
  existing artifacts, or judging anything as "redundant / duplicate / useless" — any action that
  cannot be undone. Enforces "evidence first, then act": state reproducible readings, label each
  conclusion as Verified / Inferred / Assumed, declare reversibility and your own limits, then
  execute; afterward verify instead of assuming success.
  Triggers: delete, clean up, uninstall, free space, redundant files, duplicate files, registry
  edit, stop service, overwrite, bulk move, disk cleanup, can I delete, is this a duplicate.
---

# Evidence Discipline for Irreversible Operations

## Why this skill exists

**The failure mode is not "not knowing to be careful" — it is not feeling at risk at the moment of acting.**

Four documented patterns:

- Read only a **version-info field** of an executable and declared "no digital signature" —
  while a valid signature under an authoritative CA actually existed; the real signature
  information lives behind a different query interface.
- Copied **terminal output from memory** into a table and claimed "cross-checked against
  program output" — the source file was never read in that session.
- Saw two directories that "**looked** identical" and judged one a redundant copy —
  **not a single file matched in size**.
- Looked at **blurry text** in a rendered thumbnail and announced "figures verified" —
  the original pixels were never checked.

**Common thread**: conclusions came from "I remember", "I infer", or "it looks like" —
not from "I just read a file / ran a command."

This skill does exactly one thing: **make the nature of each conclusion visible before acting.**

---

## 1. Procedure (in order, no skipping)

### Step 1 — State what will be touched

- **Absolute paths**, listed individually. No relative paths, no wildcards,
  no vague references like "those files".
- **Size and file count** for each target.
- For a batch, give the **selection rule** (e.g. "extension is X and mtime is before Y").

### Step 2 — Supply evidence, labeled

Every conclusion must answer: **"Did I read this just now, or do I remember it?"**

Three labels, all mandatory:

| Label | Meaning | Requirement |
|---|---|---|
| **[Verified]** | Obtained by running a command / reading a file this session | Must cite the file path or command |
| **[Inferred]** | Derived from verified data | Must state the derivation |
| **[Assumed]** | An unverified guess | **Never sufficient to justify acting** |

**Only [Verified] can justify a deletion.**

### Step 2b — Use reference relationships instead of "looks like"

When judging whether a file can be cleaned up, do not guess from file or directory names.
**Find out what references it:**

- Identify the generator scripts / configs still in use, and list the paths they actually read;
- Only files that appear solely in one-off scripts — whose changes have since been folded into
  the current main script — may be classified as historical intermediates;
- **Names like `old_`, `temp_`, `fix_` are not evidence.** Only reference relationships are.

### Step 3 — Judge reversibility

State explicitly **whether this can be undone**:

- Is there a backup? Where, and has it been verified?
- Recycle bin or permanent delete? (Permanent is irreversible.)
- Can it be a **move** instead of a delete? (Prefer move when possible.)
- Can a copy be made before touching the original?

### Step 4 — Declare your own limits

**State clearly what this session cannot do**, for example:

- Sufficient privileges? (Without them, protected directories and system services cannot be removed.)
- Are targets locked? (Check processes/services first, or the operation will half-fail.)
- Is a reboot required for the change to take effect?

**Never attempt first and then report the failure as the result.**

### Step 5 — Act

- One target at a time, reporting each result individually. Never collapse into "done".
- Report partial failures honestly, including leftovers. Never say "mostly complete".
- **Do not touch anything outside the requested scope** — list it and report instead.

### Step 6 — Verify (never assume success)

- **After delete/move**: rescan the target path; report remaining count and size.
- **After copy/rename**: compare source and destination hashes.
- **After uninstall**: check program directory, registry entries, services, processes, shortcuts.
- **After bulk operations**: give "N succeeded / M failed / X leftovers" in detail.
- **Collateral check**: confirm the artifacts that *should not* have been affected are intact.
  (This is the most frequently skipped step.)

**"The command didn't error" is not "the operation succeeded".**

---

## 2. Core rules

### 1. Never fill in data from memory

**Every number must come from a file read in this session.**

If the source cannot be read right now, write "not read" — **do not reconstruct it from memory.**

> This is the single most important rule here. A table filled from memory is far more dangerous
> than missing data, because it wears the appearance of "already done" and only item-by-item
> checking will reveal it.

### 2. Never treat "looks the same" as "is the same"

**Suspected duplicates must be verified with decidable quantities**: file count, individual sizes,
sampled hashes.

"Same filename", "similar directory structure", "close in size" are not criteria.

### 3. Distinguish disk space from physical memory

When a user says "running out of memory", first confirm whether they mean **disk capacity** or
**RAM usage**, then decide what to analyze.

**Deleting files does not free RAM.** Report the two separately; never conflate them.

### 4. Judging something "useless" requires verifiable grounds

- ❌ "This is an installer, no longer needed"
- ✅ "This directory holds X files totalling Y; the main program is already installed at Z
   ([Verified]); a file of the same name exists at Z with version …"

**"Looks like a temp file" is not grounds.** State which class of intermediate artifact it is
and why it can be regenerated.

### 5. Uninstalling starts with the official uninstall path

Order of preference: official uninstaller → system uninstall entry → manual removal.

- Read the registry `Uninstall` entries first to obtain `UninstallString`.
- When trying silent flags, **report which flag was used, the exit code, and whether the file
  count changed.**
- **Exit code 0 does not mean the uninstall succeeded** — recheck the file count.
  (Real case: an uninstaller with a `/S` flag returned exit code 0 while the file count was
  completely unchanged; a different flag was required to actually uninstall.)

### 6. Read permissions before editing the registry or services

- Read `Owner` and the ACL.
- If the owner is `SYSTEM` or `TrustedInstaller`, even an elevated administrator may lack the
  right to delete; the operation itself may need to change (e.g. target the 64-bit view, or take
  ownership first).
- **Never conclude "cannot be deleted" without reading the ACL** — "insufficient permission" and
  "wrong approach" are two different causes.

### 7. Separate "functionality removed" from "residual files"

Report them separately:

```
Functionality : program uninstalled / driver removed / service stopped   -> affects usage
Files         : X MB of leftover logs / Y registry keys                  -> does not affect usage
```

Let the user judge which item is worth handling.

### 8. Never delete beyond the requested scope

If the user says "delete these", delete exactly those.

If you notice other suspicious items → **list and report them**, do not delete them on your own.

---

## 3. Required verification block

Every reply involving an irreversible operation must end with:

```
-- Operation verification --
Targets      : <absolute paths>
Before       : <file count / size>
Action       : <exact command or operation>
Result       : N succeeded / M failed
After        : <file count / size from rescan>
Collateral   : <are the artifacts that should not have changed intact?>
Leftovers    : <list with impact, if any>
Unverified   : <explicitly list what was NOT verified>
```

**The "Unverified" field may not be a vague "none".**
If genuinely nothing is unverified, write "fully rechecked" and state how.

---

## 4. Pre-action checklist

- [ ] Is the target an absolute path, and have I confirmed it is the one the user means?
- [ ] Does every "useless / duplicate / redundant" judgment have [Verified] evidence?
- [ ] Did I determine cleanability from **reference relationships**, or by guessing from names?
- [ ] Have I written any [Inferred] or [Assumed] item as if it were part of the conclusion?
- [ ] Is this reversible? Can it be a move, or a backup first?
- [ ] Have I checked permissions and locks?
- [ ] If I can only finish part of it, will I report the leftovers honestly?
- [ ] How will I verify afterward — and have I decided that *before* acting?
- [ ] Is there any "should not be affected" artifact I need to check collaterally?

---

## 5. When something must be deleted but evidence is insufficient

In priority order:

1. **Turn it into a move** (reversible beats irreversible)
2. **Copy a backup, then delete the original**
3. **Delete only the part with sufficient evidence**; list the rest as pending confirmation
4. **State exactly what evidence is missing** and let the user decide

**Never substitute "the risk is probably low" for evidence.**

---

## 6. Scope and limits

This skill is a **behavioral constraint**, not a **safety checker**. It cannot:

- Replace backups
- Decide whether a file matters **to you** (only you know that)
- Stop a model that is already certain it is right

All it can do is make the nature of each conclusion and the limits of each operation visible,
**so the user has a chance to stop it before anything happens.**

**If you remember only one line:**
> Before acting, any conclusion without [Verified] backing is not a conclusion.

---

## Final goal

Every irreversible operation should satisfy three conditions:

1. **Before acting**, the user can see from the reply which conclusions are measured and which are not;
2. **While acting**, failures and partial completion are reported honestly, not packaged as "done";
3. **After acting**, the result is verified, not assumed.

The user does not need to trust me. The user only needs to be able to check me.
