# Contribution 1: Add monitoring.yml to all packages that are still without it

**Contribution Number:** 1  
**Student:** Cabdifataax Maxamuud  
**Issue:** [getsolus/packages #4121](https://github.com/getsolus/packages/issues/4121)  
**Status:** Phase II – In Review. Batch 1 PR [#9381](https://github.com/getsolus/packages/pull/9381) received a **changes-requested** review; rework in progress. Only `hunspell-fr` and `hunspell-it` have verified Anitya IDs so far; the other five Batch 1 files were reverted to templates after their IDs proved invalid. Batch 2 is authored locally but not yet committed.

---

## Why I Chose This Issue

This issue is a great entry point into open source contribution because it is well-scoped and beginner-friendly. Each sub-task (adding a `monitoring.yml` to a single package) is small and self-contained, which makes it easy to make incremental progress and get familiar with the project's structure and contribution workflow without needing deep knowledge of the codebase.

I also wanted to learn more about how Linux package management ecosystems handle security monitoring and release tracking. Working through this issue gives hands-on experience with tools like Anitya (release-monitoring.org), CPE naming conventions from the National Vulnerability Database, and the Solus packaging workflow — all of which are broadly useful skills for systems-level open source work.

---

## Understanding the Issue

### Problem Description

The Solus packages repository uses a `monitoring.yml` file in each package's directory to enable automated scanning for new upstream releases and security advisories (CVEs). A large number of packages were missing this file, meaning they were not being automatically monitored for updates or vulnerabilities. As of June 2026, approximately 16 packages remain without a `monitoring.yml` file.

### Expected Behavior

Every package directory should contain a `monitoring.yml` file alongside `package.yml`, `pspec*.xml`, and `MAINTAINERS.md`. The file should include a valid Anitya project ID for release tracking and either a CPE name for CVE tracking or a dated comment confirming that no CPE exists.

A typical `monitoring.yml` looks like this:

```yaml
releases:
  id: 8335
  rss: https://example.com/tags?format=atom
# No known CPE, checked 2025-06-23
security:
  cpe: ~
```

Or, when there is no RSS feed:

```yaml
releases:
  id: 372091
  rss: ~
# No known CPE, checked 2025-01-09
security:
  cpe: ~
```

### Current Behavior

Running the provided `find` command in the local packages repo returns a list of package directories that are missing a `monitoring.yml` file. Only ~16 packages remain unaddressed at this stage of the issue.

As of June 2026, the confirmed remaining packages (from [malfisya's gist](https://gist.github.com/malfisya/696f92c7555e043bfc0499c26aad8f58)) are:

| Package | Category |
| --- | --- |
| `hunspell-fr` | Spell checker dictionary (French) |
| `hunspell-it` | Spell checker dictionary (Italian) |
| `hunspell-pt-br` | Spell checker dictionary (Portuguese/BR) |
| `hunspell-ru` | Spell checker dictionary (Russian) |
| `hunspell-sl` | Spell checker dictionary (Slovenian) |
| `hyphen-de` | Hyphenation rules (German) |
| `hyphen-fr` | Hyphenation rules (French) |
| `iscan-data` | Epson scanner firmware data |
| `iscan` | Epson Image Scan! utility |
| `msodbcsql` | Microsoft ODBC Driver for SQL Server |
| `python-sip-4` | Legacy SIP C++/Python binding tool |
| `python-sphinx-rtd-theme` | Sphinx Read the Docs theme |
| `python-sphinx-lv2-theme` | Sphinx LV2 documentation theme |
| `python2-cairo` | Python 2 Cairo bindings (may be removed) |
| `python2-setuptools` | Python 2 setuptools (legacy) |
| `sqlheavy` | SQLite GObject wrapper (may be removed) |

> **Note:** `python2-cairo` and `sqlheavy` do not appear in the local fork's `packages/` tree, suggesting they may have already been removed or dropped upstream between the gist being written and now.

### Affected Components

- Individual package directories across the entire `getsolus/packages` repository
- The automated release monitoring and security advisory pipeline used by Solus maintainers

---

## Reproduction Process

### Environment Setup

- **OS:** Any Linux system (or WSL on Windows) with `git` and `bash` available
- **Required tools:** `go-task` (the Solus task runner — install via `go install github.com/go-task/task/v3/cmd/task@latest` or your distro's package manager)
- **Optional:** `yamllint` to validate generated YAML before committing

### Steps to Reproduce

Follow these steps exactly to confirm which packages are still missing `monitoring.yml`:

1. Fork the upstream repository on GitHub: go to [getsolus/packages](https://github.com/getsolus/packages) and click **Fork**.

2. Clone your fork locally:

   ```bash
   git clone https://github.com/<your-username>/packages.git
   cd packages
   ```

3. Run the following `find` command from the repo root to list every package directory that has no `monitoring.yml` file:

   ```bash
   find . -maxdepth 3 -mindepth 3 -name "package.yml" \
     | sed 's|/package\.yml||' \
     | while read d; do [ ! -f "$d/monitoring.yml" ] && echo "$d"; done \
     | sort
   ```

   > **What this does:** walks every `packages/<letter>/<name>/` directory, checks for the presence of `monitoring.yml`, and prints only those directories where the file is absent.

4. Compare the output against the [reference gist](https://gist.github.com/malfisya/696f92c7555e043bfc0499c26aad8f58). As of June 2026 you should see roughly 14 packages (the same list in the table above — `python2-cairo` and `sqlheavy` are no longer in the tree).

5. Navigate into one of the listed package directories and scaffold the template:

   ```bash
   cd packages/h/hunspell-fr
   go-task add-monitoring
   cat monitoring.yml        # inspect the generated skeleton
   ```

   You will see a YAML file with placeholder `id`, `rss`, and `cpe` fields that must be filled in before committing.

### Reproduction Evidence

- **Reproduction command output:** confirms ~14 actionable packages still missing the file (see table in Current Behavior above)
- **Full list of originally affected packages:** [malfisya's gist](https://gist.github.com/malfisya/696f92c7555e043bfc0499c26aad8f58)
- **Working branch in fork:** [Abdifatah2023/packages — add-monitoring-yml-batch1](https://github.com/Abdifatah2023/packages/tree/add-monitoring-yml-batch1)

---

## Solution Approach

### Analysis

The root cause is simply that `monitoring.yml` is a relatively new requirement and was not retroactively added to existing packages when it was introduced. There is no bug in any single package — this is a coverage gap that is being filled incrementally by contributors. The bulk of the work has already been done by the community; only ~16 packages remain.

### Proposed Solution

Pick a batch of 5–10 packages from the remaining list, and for each one:

1. Run `go-task add-monitoring` inside the package directory to scaffold the template
2. Look up the package on release-monitoring.org to find its Anitya ID
3. Add the correct `id` (and `rss` feed if one exists, otherwise `rss: ~`) under the `releases` section
4. Search for a CPE name using the `cpe-guesser` API or `cpesearch` helper
5. Fill in the `cpe` field, or add a `# No known CPE, checked YYYY-MM-DD` comment and set `cpe: ~`
6. Commit per package and open one PR at a time (5–10 packages per PR per maintainer guidance)

### Implementation Plan

My approach is to add `monitoring.yml` files in three small batches, one PR per batch, so each PR stays within the 5–10 package limit set by maintainers:

- **Batch 1 (PR #1):** the seven `hunspell-*` / `hyphen-*` dictionary packages — these share a similar upstream pattern (language-specific projects), so the Anitya lookups are predictable and CPEs are unlikely, making them a fast, low-risk first batch.
- **Batch 2 (PR #2):** four Python packages (`python-sip-4`, `python-sphinx-rtd-theme`, `python-sphinx-lv2-theme`, `python2-setuptools`) — well-known upstream projects that are almost certainly already tracked on Anitya.
- **Batch 3 (PR #3):** three proprietary/vendor packages (`iscan-data`, `iscan`, `msodbcsql`) — these require extra CPE research (Epson and Microsoft vendor namespaces) so they are handled last once the workflow is proven.

For each package: scaffold with `go-task add-monitoring` → look up the Anitya project ID → check NVD for a CPE → fill in the file → commit with message `<package>: Add monitoring.yaml` → open PR referencing #4121.

---

Using UMPIRE framework (adapted):

**Understand:** Packages are missing `monitoring.yml` files that link them to upstream release tracking and CVE databases. Each file must be individually researched and filled out. Only ~16 remain.

**Match:** Existing `monitoring.yml` files in the repo (e.g., `hatari/monitoring.yml`, `passt/monitoring.yml`) serve as reference examples for correct formatting and field usage.

**Plan:** Batch the remaining 14 packages as described in **Implementation Plan** above (dictionary → Python → proprietary). For each package: scaffold, look up the Anitya ID on release-monitoring.org, check NVD for a CPE, fill in the file, and commit it on its own. Open one PR per batch referencing #4121, waiting for review before opening the next.

1. ~~Run the `find` command to confirm the current list of remaining packages~~ ✓ Confirmed: 14 actionable packages (`python2-cairo` and `sqlheavy` are gone from the repo)

**Implement:** Batch 1 (7 `hunspell-*`/`hyphen-*` packages) committed on `add-monitoring-yml-batch1` and submitted as PR [#9381](https://github.com/getsolus/packages/pull/9381). The PR then received a **changes-requested** review (see **Pull Request → Maintainer Review**), which surfaced that several of the Anitya IDs did not actually resolve to useful upstream release data. Five of the seven Batch 1 files were reverted to their template skeletons pending real upstream sources; only `hunspell-fr` and `hunspell-it` remain filled in. Batch 2 (7 Python/proprietary packages) is authored locally but held back and not yet committed. See **Code Changes** for branch and per-package commit links.

**Review:** Each file was checked against the schema in the reference examples and validated as valid YAML. The critical gap caught in review: passing YAML validation is *not* the same as a working Anitya ID — an `id` can be syntactically fine yet point at a project with no usable release feed. Anitya IDs must be verified to return real version info at `https://release-monitoring.org/project/<id>/`, which is now the corrective focus. See **Testing Strategy**.

**Evaluate:** After each PR merges, confirm the packages no longer appear in the coverage `find` command output. As of this update, PR #9381 has **not** merged — it is open with requested changes.

---

## Testing Strategy

### Coverage Check — Verify No Package Is Missing a Monitoring File

After authoring the new files, this command was used to confirm no package directory is still missing a monitoring file (accepting either extension):

```bash
find . -maxdepth 3 -mindepth 3 -name "package.yml" \
  | sed 's|/package\.yml||' \
  | while read d; do
      [ ! -f "$d/monitoring.yaml" ] && [ ! -f "$d/monitoring.yml" ] && echo "MISSING: $d"
    done | sort
```

**Result:** No output — every package directory now contains a monitoring file.

### ID Audit — Verify All `monitoring.yaml` Files Contain a Numerical ID

A Python script was used to scan every `monitoring.yaml` for an `id:` field containing a number:

```python
import os, re

base = 'packages/'
for letter in sorted(os.listdir(base)):
    for pkg in sorted(os.listdir(os.path.join(base, letter))):
        mon = os.path.join(base, letter, pkg, 'monitoring.yaml')
        if not os.path.isfile(mon):
            continue
        with open(mon) as f:
            content = f.read()
        if not re.search(r'id:\s*[0-9]+', content):
            print(f'{letter}/{pkg}')
```

**Result:** 112 packages have `id: ~` (null). Each was verified to contain a maintainer comment confirming this is intentional (e.g., `# This package will never benefit from a monitoring.yaml` referencing issue #4533). No corrective action required.

### YAML / Schema Validation

Every new `monitoring.yaml` (both batches) was parsed with Python's `yaml.safe_load` to confirm it is syntactically valid and exposes a numeric `releases.id`:

```bash
for f in $(git ls-files -o --exclude-standard '*/monitoring.yaml') packages/h/*/monitoring.yaml; do
  python -c "import yaml,sys; d=yaml.safe_load(open('$f')); \
    assert isinstance(d['releases']['id'], int), 'non-numeric id'; print('OK', '$f', d['releases']['id'])"
done
```

**Result (initial):** all files parsed as valid YAML. **Important correction:** valid YAML with a numeric `id` was treated as "done" too early. Maintainer review then showed that several of the numeric IDs — particularly the sequential `390xxx` IDs, some of which were Anitya projects I had created myself during this work — did **not** return useful upstream release data. As a result, the current verified state is narrower than this check implied:

- **Verified, working Anitya IDs:** `hunspell-fr` → 130209 (grammalecte), `hunspell-it` → 390712. Batch 2's pre-existing project IDs (`python-sip-4` → 13626, `python-sphinx-rtd-theme` → 232897, `python-sphinx-lv2-theme` → 267121, `python2-setuptools` → 4021) also resolve, but Batch 2 is not yet committed or submitted.
- **Reverted to template (`id: ~`):** `hunspell-pt-br`, `hunspell-ru`, `hunspell-sl`, `hyphen-de`, `hyphen-fr` — their IDs did not resolve to usable release feeds, so they were rolled back rather than shipped with bad data.

Lesson: a numeric `id` is necessary but not sufficient — the ID must actually point at a project Anitya can scrape for versions.

### Validation Checklist

- [x] Every authored `monitoring.yaml` is syntactically valid YAML (verified via `yaml.safe_load`)
- [x] Packages with `id: ~` elsewhere in the repo confirmed intentional (maintainer comment present)
- [x] `hunspell-fr` (130209) and `hunspell-it` (390712) IDs confirmed to resolve at `https://release-monitoring.org/project/<id>/` with real version data
- [ ] **Blocking:** confirm the remaining Batch 1 IDs resolve to a usable release feed — currently failing, so those five files are reverted to templates
- [ ] Locate valid upstream release sources (or an accepted fallback) for `hunspell-pt-br`, `hunspell-ru`, `hunspell-sl`, `hyphen-de`, `hyphen-fr`
- [ ] Re-verify Batch 2 IDs and CPEs, then commit and open a second PR (only after #9381 is resolved)

### Manual Testing

Anitya IDs were looked up on release-monitoring.org during authoring. This is where the process broke down: syntactic lookups were treated as confirmation too quickly, and several IDs (including some I created on Anitya myself) turned out not to expose real releases. Manual re-verification — actually opening each `https://release-monitoring.org/project/<id>/` page and confirming it shows recent versions — is now the standard before an ID is committed. CPE names were checked against the NVD CPE browser; none of these packages have a published CPE, so each is recorded as `cpe: ~` with a dated comment per the schema.

---

## Implementation Notes

### Week 1 Progress

Posted an introductory comment on issue #4121 explaining my background (CITE student at TCU / CodePath AI Open-Source program) and planned workflow. Received a reply from project member `davidjharder` with specific guidance on batch sizes and PR workflow. Currently selecting the first batch of packages from the remaining ~16 and preparing to open a first PR.

### Week 2 Progress — Repository Exploration

Cloned the upstream `getsolus/packages` repository locally and explored the structure to prepare for implementation.

**Reference `monitoring.yml` files confirmed in the repo:**

Three files currently exist and serve as ground-truth examples:

- `packages/h/hatari/monitoring.yml` — uses Anitya ID 8335, includes an RSS atom feed, no CPE (checked 2025-06-23)
- `packages/p/passt/monitoring.yml` — uses Anitya ID 372091, RSS set to `~` (no feed), no CPE (checked 2025-01-09)
- `packages/s/scd2html/monitoring.yml` — uses Anitya ID 316146, includes an RSS feed from sourcehut, no CPE (checked 2025-04-30)

These confirm the schema shown in the "Understanding the Issue" section above is accurate.

**Key observations on the 16 remaining packages:**

- The `hunspell-*` and `hyphen-*` packages are dictionary/hyphenation data files — their upstream sources are language-specific projects (e.g., grammalecte.net for French). Anitya IDs will need individual lookup; CPEs are unlikely.
- `iscan` sources from Epson's Linux scanner package hosting (`epson.net`). Proprietary distribution; CPE may exist under `epson` vendor.
- `msodbcsql` sources from `packages.microsoft.com` (Microsoft ODBC Driver v17). Anitya is unlikely to track this; CPE likely exists under `microsoft` vendor.
- `python-sip-4` sources from `riverbankcomputing.com` — very likely tracked on Anitya (well-known Python/C++ binding tool).
- `python2-cairo` and `sqlheavy` are **not present** in the local clone's package tree, suggesting they were dropped from the Solus repo before a monitoring file was ever needed. They can be skipped.

### Week 3 Progress — Implementation

All 14 actionable packages from the remaining list have now had a `monitoring.yaml` file authored. None of these packages had any prior monitoring file in the repository (confirmed via `git log` on each package directory) — every file in this contribution is newly created, following the repo's dominant `.yaml` convention (4834 `.yaml` files vs. 3 legacy `.yml`).

Work was split into two batches per maintainer guidance (5–10 packages per PR):

- **Batch 1 — committed and submitted as PR [#9381](https://github.com/getsolus/packages/pull/9381) (7 packages):** the dictionary/hyphenation set `hunspell-fr` (130209), `hunspell-it` (390712), `hunspell-pt-br` (390759), `hunspell-ru` (390761), `hunspell-sl` (390767), `hyphen-de` (390754), `hyphen-fr` (390769). Each was scaffolded, given its looked-up Anitya `id`, set to `rss: ~` where no feed exists, and marked `cpe: ~` with a dated `# No known CPE` comment. One commit per package (see Code Changes).
- **Batch 2 — authored, pending commit (7 packages):** `iscan`, `iscan-data`, `msodbcsql`, `python-sip-4`, `python-sphinx-lv2-theme`, `python-sphinx-rtd-theme`, `python2-setuptools`. New `monitoring.yaml` files exist in the working tree (currently untracked) with Anitya IDs filled in (e.g. `iscan` → id `390774`, `python-sip-4` → id `13626`), `cpe: ~`, and a `# No known CPE, checked 2026-06-19` note.

A secondary audit checked every `monitoring.yaml` in the repository for a numerical Anitya ID. 112 packages were found with `id: ~` (YAML null) — this is intentional: each contains a comment (e.g. referencing issue #4533) confirming the package will never benefit from automated monitoring. No corrective action was needed.

### Week 4 Progress — Review & Rework

PR #9381 received a **changes-requested** review from maintainer `davidjharder`. Two problems drove it:

1. **Invalid Anitya IDs.** The maintainer asked, *"did you create some of these release-monitoring entries as well? Some don't work."* This was correct — several of the `390xxx` IDs pointed at Anitya projects (some I had created myself) that did not expose real upstream releases. Passing YAML validation had masked this.
2. **PR hygiene / AI transparency.** The PR had accidentally dropped the pull-request template and included a stray `contribution_readme.md` committed into the packages repo itself (separate from *this* portfolio write-up). Solus has no formal AI policy yet, so the maintainer set an explicit bar: contributors must show they manually verified any AI output and must disclose, in their own words, where and how AI was used.

**Corrective actions taken / in progress:**

- Removed the stray `contribution_readme.md` from the packages PR (commit `672a4cce`) and restored the standard PR template, including the licensing agreement checkbox.
- Reverted the five unverifiable Batch 1 files (`hunspell-pt-br`, `hunspell-ru`, `hunspell-sl`, `hyphen-de`, `hyphen-fr`) to `id: ~` templates rather than ship bad IDs; kept `hunspell-fr` and `hunspell-it`, whose IDs were confirmed to resolve.
- Posted an AI-usage explanation on the PR (see **AI Usage Disclosure**).
- Began hunting for real upstream release sources for the reverted packages. Two were re-established via upstream feeds; the remaining five are hard because several `hunspell-*`/`hyphen-*` sources are unusual and lack a scrapeable release/tags feed — which the maintainer confirmed is *why* these were among the last packages without monitoring files. An open question is pending with the maintainer: whether the Fedora Updates System can serve as an accepted fallback upstream source for Anitya when no native feed exists.

### Code Changes

- **Active branch:** [Abdifatah2023/packages — add-monitoring-yml-batch1](https://github.com/Abdifatah2023/packages/tree/add-monitoring-yml-batch1)
- **Key commits (hunspell/hyphen batch):**
  - [`5878827`](https://github.com/Abdifatah2023/packages/commit/5878827fbb) — `hunspell-fr: Add monitoring.yaml`
  - [`151c7ea`](https://github.com/Abdifatah2023/packages/commit/151c7eaf9a) — `hunspell-it: Add monitoring.yaml`
  - [`41430d0`](https://github.com/Abdifatah2023/packages/commit/41430d0e66) — `hunspell-pt-br: Add monitoring.yaml`
  - [`8a85523`](https://github.com/Abdifatah2023/packages/commit/8a85523e58) — `hunspell-ru: Add monitoring.yaml`
  - [`cc3ef05`](https://github.com/Abdifatah2023/packages/commit/cc3ef05a09) — `hunspell-sl: Add monitoring.yaml`
  - [`6f99157`](https://github.com/Abdifatah2023/packages/commit/6f99157ad1) — `hyphen-de: Add monitoring.yaml`
  - [`bc1505f`](https://github.com/Abdifatah2023/packages/commit/bc1505ffbb) — `hyphen-fr: Add monitoring.yaml`
- **Pending (Batch 2, not yet committed):** `iscan`, `iscan-data`, `msodbcsql`, `python-sip-4`, `python-sphinx-lv2-theme`, `python-sphinx-rtd-theme`, `python2-setuptools` — new `monitoring.yaml` files authored in the working tree (untracked). Held back until #9381 is resolved and each ID has been re-verified against release-monitoring.org under the corrected process.
- **Pull request (Batch 1):** [getsolus/packages #9381](https://github.com/getsolus/packages/pull/9381) — opened against upstream, references #4121.
- **Approach decisions:** Following maintainer guidance — one PR at a time, 5–10 packages per PR, each package as its own commit. The `hunspell-*` / `hyphen-*` dictionary packages formed the first batch; the Python and proprietary packages form the second.

---

## Pull Request

**PR Link (Batch 1):** [getsolus/packages #9381](https://github.com/getsolus/packages/pull/9381)

**Summary:** Adds `monitoring.yaml` to 7 spell-checker and hyphenation dictionary packages that were missing it, enabling automated release and CVE monitoring. Part of #4121.

The table below shows the *originally submitted* IDs and their status after maintainer review. Only the first two are currently confirmed working; the rest were reverted to `id: ~` templates.

| Package | Anitya ID (submitted) | CPE | Current status |
| --- | --- | --- | --- |
| `hunspell-fr` | 130209 (grammalecte) | none | ✅ Verified, kept |
| `hunspell-it` | 390712 | none | ✅ Verified, kept |
| `hunspell-pt-br` | 390759 | none | ↩️ Reverted to template (ID did not resolve) |
| `hunspell-ru` | 390761 | none | ↩️ Reverted to template (ID did not resolve) |
| `hunspell-sl` | 390767 | none | ↩️ Reverted to template (ID did not resolve) |
| `hyphen-de` | 390754 | none | ↩️ Reverted to template (ID did not resolve) |
| `hyphen-fr` | 390769 | none | ↩️ Reverted to template (ID did not resolve) |

**Test plan (as originally posted on the PR):**

- Verified each `monitoring.yaml` is valid YAML
- Confirmed all Anitya IDs resolve at [release-monitoring.org](https://release-monitoring.org)
- Checked the NVD CPE database for each package; none have CVE entries

> **Correction:** the second bullet did not hold up. "Resolve" was checked too loosely — several IDs pointed at Anitya projects with no usable release data, which the maintainer caught. Only `hunspell-fr` and `hunspell-it` genuinely resolve to real version info. See **Maintainer Review** below.

**Batch 2** (`iscan`, `iscan-data`, `msodbcsql`, and the four Python packages) is authored locally but deliberately not yet committed — it will only follow once #9381 is resolved and the verification process has been corrected.

**My Comment on the Issue (2026-06-11):**

> Hi! I am a CITE student at TCU and currently part of the CodePath AI Open-Source program. I'd like to contribute to this issue. I'm planning to add `monitoring.yml` files to a batch of packages from the list in the gist.
>
> I'll start with a small set to make sure I've got the workflow right (finding the correct Anitya ID on release-monitoring.org, checking for CPE names, and using `go-task add-monitoring`), then expand from there. I'll open a PR once I have a first batch ready for review.
>
> Let me know if there are any packages I should prioritize or avoid, or if there's a preferred batch size for PRs. Thanks!

**Maintainer Response — davidjharder (2026-06-11):**

> Hi Abdifatah, you are welcome to try a few batches. but there is not much left to do on this particular task.
> For easier review, please work on one PR at a time and limit PRs to 5-10 packages.
> We (the Solus project) have not hammered out AI guidelines yet, but all contributions must be fully understood by the PR author.

**Key takeaways from maintainer feedback:**

- Only ~16 packages remain — the task is nearly complete
- Submit one PR at a time, 5–10 packages per PR
- Every contribution must be personally understood by the author, regardless of tooling used

### Maintainer Review — davidjharder (changes requested)

After the PR was opened, the maintainer first asked whether I had created some of the release-monitoring entries myself, noting *"some don't work,"* then left a formal **changes-requested** review:

> At this moment Solus does not have guidelines for AI usage; so I will substitute my own: If you want human review and approval you must demonstrate that you, the person making this PR, have put some effort into manually verifying any AI outputs. You should also say, preferably in your own words, where and how AI was used.
>
> Changes required:
> - Remove "contribution readme"
> - Verify the provided anitya IDs are working (return useful version info)
> - Restore the PR template, including the licensing portion
> - Briefly explain your use of AI

**My response and follow-up:** I removed the stray contribution readme and restored the PR template with the licensing checkbox, explained my AI use (below), and acknowledged that some of my Anitya projects did not reflect correct releases. I asked the maintainer for guidance on finding valid upstream release sources for the difficult dictionary packages. His reply: the sources for some `hunspell` packages are genuinely unusual — which is largely *why* they still lacked monitoring files — and there is no shortcut beyond trial and error against existing Anitya IDs as examples. My current open question to him is whether the Fedora Updates System is an acceptable fallback upstream for the packages with no native release feed.

**Status:** Batch 1 PR [#9381](https://github.com/getsolus/packages/pull/9381) is **open with changes requested**. `hunspell-fr` and `hunspell-it` are verified and kept; the other five files are reverted to templates while I search for valid sources. Batch 2 authored locally, not yet committed.

---

## AI Usage Disclosure

Per the maintainer's requirement — and because transparency is the right default when contributing to a project that has not yet set its AI policy — here is an honest account of where and how I used AI on this contribution.

**Where AI was used:**

- **Understanding the project.** At the start I used an AI coding assistant (Claude / Claude Code) to explore the `getsolus/packages` repository and explain its structure, the `package.yml`/`monitoring.yaml` layout, the build tooling, and the contribution conventions, so I could get oriented in a large unfamiliar monorepo quickly.
- **Authoring the monitoring files.** I used AI to help scaffold and fill in the `monitoring.yaml` files — including help researching candidate Anitya IDs and writing regular expressions for the version-scraping URLs used to set up release-monitoring.org projects.
- **This write-up.** AI assisted in drafting and updating this portfolio document, including this disclosure. The facts, decisions, and the account of what went wrong are mine; AI helped organize and phrase them.

**How I verified the output (and where I fell short):**

- Anitya IDs were meant to be checked on release-monitoring.org and CPEs against the NVD. My honest failure was verifying too shallowly: I treated a syntactically valid file and an existing project page as "working," when several of the projects I set up did not actually expose real releases. The maintainer caught this.
- The corrective process I now follow: open each `https://release-monitoring.org/project/<id>/` page and confirm it shows recent version data *before* committing an ID; when it does not, leave the file as an `id: ~` template rather than ship a placeholder that looks complete. `hunspell-fr` and `hunspell-it` passed this bar; the other five did not and were rolled back.

**Takeaway:** AI was useful for orientation and boilerplate, but the release-monitoring work requires human judgment about whether an upstream source is real and scrapeable — exactly the kind of verification the maintainer insisted on, and the part I under-did the first time.

---

## Learnings & Reflections

### Technical Skills Gained

- Understanding of how Linux distributions track upstream releases and security advisories
- Familiarity with Anitya / release-monitoring.org and the CPE naming standard
- Experience navigating a large monorepo and contributing at scale
- Understanding of the `monitoring.yml` schema used by Solus (releases ID, rss feed, security CPE)

### Challenges Overcome

- Learning the correct `monitoring.yml` format by reading existing files in the repo (e.g., `hatari/monitoring.yml`, `passt/monitoring.yml`) rather than relying solely on documentation
- Discovering that some packages had the file saved as `.yml` instead of the required `.yaml` extension — a silent typo that caused them to appear missing even though the content was correct
- Scoping effort correctly given that only ~16 packages remain rather than the original ~3,000+

### What I'd Do Differently Next Time

- **Verify Anitya IDs properly before committing, not after.** The biggest lesson: a valid-looking `monitoring.yaml` is not a working one. I'd confirm each ID resolves to real upstream release data on release-monitoring.org before it ever reaches a commit, and leave anything unconfirmed as a template.
- **Don't over-rely on AI for the judgment part.** AI was fine for orientation and boilerplate, but deciding whether an upstream source is genuine and scrapeable is human work — skipping that is what produced the bad IDs.
- **Keep the PR clean from the start.** Never commit personal/portfolio files into the target repo, and double-check the PR template and licensing checkbox are intact before opening.
- **Pick the tractable packages first and be honest about the hard ones.** Several of these packages are unmonitored precisely because their upstreams have no release feed; I'd flag those as "no viable source yet" early rather than force an ID.

---

## Resources Used

- [Solus monitoring.yml documentation](https://help.getsol.us/docs/packaging/monitoring.yml/)
- [GitHub Issue #4121](https://github.com/getsolus/packages/issues/4121)
- [Gist: packages missing monitoring.yml](https://gist.github.com/malfisya/696f92c7555e043bfc0499c26aad8f58)
- [release-monitoring.org (Anitya)](https://release-monitoring.org/)
- [CPE Guesser API](https://cpe-guesser.cve-search.org/search)
- [National Vulnerability Database CPE Search](https://nvd.nist.gov/products/cpe/search)
