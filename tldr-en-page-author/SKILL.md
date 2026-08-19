---
name: tldr-en-page-author
description: Standardize the end-to-end contribution workflow for authoring missing English pages in tldr-pages, including eligibility and duplicate checks, platform resolution, evidence-based command research, deterministic validation, human review, local commit, fork push, and browser PR handoff. Use when the user invokes this skill with one or more command names and wants to add new English pages in a local tldr-pages/tldr clone. Do not use for editing existing pages or adding translations.
---

# Author Missing English tldr Pages

Treat one invocation as one authoring batch. Keep every user-provided command in the same branch, commit, and PR when repository limits and page relationships permit. Do not silently split or shrink the batch.

## Scope

- Add only new English pages under `pages`.
- Support ordinary command pages, subcommand pages, alias pages, grouped-command references, disambiguation pages, and justified platform variants.
- Do not edit an existing English page or create translations.
- End after the PR is created and verified.
- Do not track CI, handle review comments, monitor merging, or clean branches in v1.
- Run from the root of a local `tldr-pages/tldr` clone.

Use this state flow:

```text
REQUESTED
  -> REPOSITORY_READY
  -> RESEARCHED
  -> READY_FOR_AUTHORING
  -> DRAFTED
  -> VALIDATED
  -> CONTENT_APPROVED
  -> LOCAL_FINALIZATION_APPROVED
  -> LOCALLY_FINALIZED
  -> REMOTE_PREPARATION_APPROVED
  -> PR_READY_FOR_REVIEW
  -> PR_CREATED
```

## Global Controls

- Treat the ordered command set, command identity, page type, and resolved target paths as the batch identity. If any of them changes, return to research and rerun all affected preflight checks.
- Aggregate all blockers found in a phase. Explain each cause, recommend a concrete resolution, and then ask the user what to do.
- Never invent command behavior, silently drop a command, choose an ambiguous platform, install or upgrade the documented software, stash work, overwrite a branch, bypass hooks, force-push, delete a branch, or rewrite commit history.
- Do not run an example that can modify real user data, devices, accounts, services, or remote state merely to verify it. Prefer official documentation, built-in help, source documentation, and isolated disposable environments.
- Bind every approval to the exact files, content, and plan shown to the user. Any page-content change invalidates `CONTENT_APPROVED`. Any material commit or remote-plan change invalidates its corresponding approval.
- Keep `CONTENT_APPROVED`, `LOCAL_FINALIZATION_APPROVED`, and `REMOTE_PREPARATION_APPROVED` distinct.
- Resolve repository paths relative to the repository root and bundled resources relative to this `SKILL.md`.

## Parse the User Input

- Accept command names; do not require the user to supply platform names, page paths, versions, or documentation links.
- Accept one command or a comma-, Chinese-enumeration-comma-, newline-, or natural-language-separated list.
- Preserve spaces inside subcommands. Treat `git commit` and `git-commit` as equivalent lookup forms.
- Preserve the user's command order throughout the batch.
- If command boundaries or command identity are ambiguous, show the parsed list, explain the ambiguity, recommend an interpretation, and ask the user before research proceeds.

## Phase 1: Repository Preflight

Enter with `REQUESTED`. Complete these checks without creating a branch:

1. Read the current `.github/PULL_REQUEST_TEMPLATE.md` and extract its new-page limit. The current expected limit is 5, but the repository document is authoritative.
2. Require a clean worktree using `git status --porcelain=v1 --untracked-files=all`. If dirty, list every change and recommend that the user save or finish it. Do not stash, commit, or delete it.
3. Inspect every remote URL. Identify `tldr-pages/tldr` by normalized URL, never by assuming the remote is named `upstream` or `origin`.
4. Run `gh auth status`. If unauthenticated, recommend `gh auth login` and pause. Never handle or print credentials.
5. Fetch the official upstream `main` explicitly.
6. Inspect the requested command names against the fetched ref:

   ```text
   python <skill-dir>/scripts/validate_batch.py inspect \
     --repo <repo-root> \
     --ref <official-remote>/main \
     --json \
     <commands...>
   ```

7. Report all existing English paths for each normalized name. An existing page normally means the request is out of scope, but it may reveal a justified missing platform variant or name collision that must be resolved during research. Never overwrite the existing page.
8. Confirm that the requested batch does not already exceed the current new-page limit. Account for any required base, alias target, or disambiguation page after research and check the limit again.

Report the repository checks, normalized inputs, existing matches, and `REPOSITORY_READY`.

## Phase 2: Research and Resolve the Batch

Enter only with `REPOSITORY_READY`.

Read [references/en-page-authoring-rules.md](references/en-page-authoring-rules.md) completely. Then read every current repository section it routes. Current repository rules outrank the bundled summary.

For every command, build a research record containing:

- Exact command identity, owning project or operating system, and invocation spelling.
- Evidence that the project satisfies the repository's age/notability requirement when that requirement applies.
- Page type, normalized filename, heading spelling, and platform directory, with evidence for platform availability.
- A direct English usage-documentation URL suitable for `More information`, or the repository-supported fallback.
- The documented command version when known.
- The intended examples in priority order, with primary evidence for every subcommand, option, positional argument, and behavior claim.
- Related English pages consulted for style, page granularity, and shared terminology.
- Any unresolved facts, unsupported examples, safety limitations, or version/platform differences.

Use official documentation, built-in help/man output, and upstream source documentation as command-truth sources. Existing tldr pages are style references, not proof that syntax or behavior is correct. Safely execute examples only when doing so adds evidence and can be isolated from user data and external state. Do not install the command or broaden permissions without explicit authorization.

Resolve subcommands, aliases, grouping, platform variants, and name collisions according to current repository rules. If a subcommand requires a missing base page, include that dependency in the batch or pause for the user to choose a smaller valid scope. If reliable evidence cannot determine identity, eligibility, platform, or syntax, aggregate the gaps and ask the user instead of drafting.

After resolving exact target paths:

1. Rerun the current new-page-limit check with all required pages.
2. Query open upstream PRs with file data:

   ```text
   gh pr list --repo tldr-pages/tldr --state open --limit 1000 \
     --json number,title,url,files
   ```

   Compare actual changed file paths with every target path. Do not infer duplicates from PR titles alone.
3. Require every target path to be absent from the fetched upstream ref.
4. Create one branch from the fetched upstream `main` only after all research and duplicate checks pass.

Name the branch `add/<batch-slug>`:

- Normalize subcommands to hyphenated page names, such as `git commit` -> `git-commit`.
- Join multiple commands with `_`, such as `add/jq_git-commit_git-add`.
- Do not add a date, hash, or random suffix preemptively.
- If the local name already exists, do not reuse or delete it. Recommend a readable numeric suffix and ask the user.

Report the research records, exact target paths, open-PR result, branch name, known command versions, unresolved limitations, `RESEARCHED`, and then `READY_FOR_AUTHORING`.

## Phase 3: Author the Whole Batch

Enter only with `READY_FOR_AUTHORING`.

Author every page from the approved research records and the current repository rules. Apply these boundaries:

- Prefer approximately five high-value examples and never exceed the current maximum of eight.
- Order examples from simple and common to more advanced. Keep help and version examples last when they are useful enough to include.
- Use imperative descriptions, long options when supported and appropriate, standard placeholders, and the repository's required heading order.
- Make every example traceable to the recorded evidence. Remove or flag an example whose syntax or behavior cannot be supported.
- Represent dangerous targets with conspicuous placeholders and include `sudo` when the command genuinely requires it; never weaken safety merely to produce a shorter example.
- Reuse established style and shared wording from the closest related English pages, but do not copy command-specific behavior without independent evidence.
- For an alias page, use `python scripts/set-alias-page.py -p <platform>/<alias-command> -l en` without `--stage`, then verify the generated page and alias target.
- Finish the complete batch before validation.

Record meaningful editorial choices and any evidence limitations that the user should review. Report the created files, example-to-evidence summary, limitations, and `DRAFTED`.

## Phase 4: Run Hard Validation

Enter only with `DRAFTED`. Do not use hard validation to decide whether the selected examples are useful or whether behavior claims are true; those remain evidence and human-review concerns.

Run all of the following against the exact target paths:

1. Run the project-local Markdown linter.
2. Run the project-local `tldr-lint` against each exact English page without translation ignore codes.
3. Run the bundled validator with the fetched upstream ref and scope checking:

   ```text
   python <skill-dir>/scripts/validate_batch.py validate \
     --repo <repo-root> \
     --ref <official-remote>/main \
     --check-scope \
     --json \
     --page pages/<platform>/<command>.md \
     [--page ...]
   ```

The bundled validator checks target paths, platform directories, upstream absence, lowercase filenames, heading/filename mapping, page structure, example/command counts, example limits, trailing whitespace, final newlines, duplicate targets, and exact worktree scope. It is read-only.

Automatically fix only deterministic formatting or structural failures. After any fix, rerun the entire phase. Never change an option, argument, behavior claim, target platform, or example selection as an automatic lint fix. If a failure cannot be fixed reliably, aggregate it, recommend a resolution, and ask the user.

Report the hard-check summary, automatic fixes, research limitations, and `VALIDATED`.

## Phase 5: Human Review and Iteration

Enter only with `VALIDATED`. Report exactly:

- Batch commands and new-file list.
- Command versions, primary sources, and concise example-to-evidence summary.
- Unresolved limitations and meaningful editorial choices.
- Hard-validation summary.
- An explicit statement that nothing has been committed, pushed, or submitted as a PR.

Remind the user that the repository requires genuine human review of AI-assisted pages and ask them to inspect the complete local pages. Do not claim human review merely because automated checks passed.

For every requested change:

1. Edit the affected page.
2. Invalidate the prior evidence record if the change affects syntax or behavior.
3. Re-research affected claims when needed.
4. Rerun the entire hard-validation phase.
5. Present the complete review package again.

Never infer approval from silence or an ambiguous reply. On explicit approval of the exact content after disclosure, report `CONTENT_APPROVED`. Do not commit yet.

## Phase 6: Local Finalization

Enter only with `CONTENT_APPROVED`.

1. Read the current `CONTRIBUTING.md` section “Commit message and PR title.”
2. Draft one commit message for the batch. Use current repository rules and recent accepted English page commits only to resolve uncovered ambiguity.
3. Show the exact target files and proposed commit message. Ask for explicit local-finalization authorization.
4. After approval, report `LOCAL_FINALIZATION_APPROVED` and rerun the entire hard-validation phase.
5. If any page content changes during final validation, invalidate content approval and return to Phase 4 and human review.
6. Confirm that the worktree contains only the expected new target files.
7. Stage exact paths with `git add -- <target-pages...>`; never use a broad add.
8. Run `git diff --cached --check`, inspect `git diff --cached --name-status`, and inspect the complete staged content.
9. Create exactly one commit with the approved message. Do not use `--no-verify`.
10. Verify the new commit's hash, message, files, content, and post-commit worktree status.

If a hook or commit fails, report the original error and staged state. Do not reset, delete files, bypass the hook, or rewrite history. If a successful commit is later found wrong, stop and ask the user; do not amend or reset automatically.

On success, report only the commit hash/message, committed files, final validation and worktree result, and an explicit statement that nothing has been pushed or submitted. Report `LOCALLY_FINALIZED`.

## Phase 7: Remote Preparation and PR Handoff

Enter only with `LOCALLY_FINALIZED`. Do not repeat research by default, but do recheck any facts that have become stale or were changed after approval.

1. Use `gh api user` to identify the current account.
2. Reuse the official upstream found in preflight. Identify the user fork by URL, fork relationship, and permission with `gh repo view`; do not assume `origin` is the fork.
3. Require a unique writable fork remote. Never push to the official upstream.
4. Read the current commit/PR title rule and `.github/PULL_REQUEST_TEMPLATE.md`.
5. Default the PR title to the commit message. Preserve the complete current PR template.
6. Fill in the documented command version when known. Check only boxes supported by actual evidence. Check the human-review item only because the exact content reached `CONTENT_APPROVED`. Do not invent versions or issue references. Use `Closes` only when the PR fully resolves a known issue.
7. Show the fork remote/repository, remote branch, PR base/head, title, and complete body. Ask for explicit remote-preparation authorization.
8. After approval, report `REMOTE_PREPARATION_APPROVED` and push with `git push -u <fork-remote> HEAD`. Never force-push.
9. Open the prefilled form without creating the PR:

   ```text
   gh pr create --repo tldr-pages/tldr \
     --base main \
     --head <user>:<branch> \
     --title <title> \
     --body-file <temporary-body-file> \
     --web
   ```

10. Stop at `PR_READY_FOR_REVIEW`. Ask the user to inspect the full diff, base/head, title, version, body, and checklist in the browser.
11. Let the user click “Create pull request” by default. If the user explicitly asks the agent to submit, reread the visible base/head, title, and body first and submit only if they still match the authorized plan.
12. After creation, use `gh pr view` or `gh pr list` to verify the PR number, title, URL, base/head, and state.

If push fails, do not force. If a remote branch already exists or the push is non-fast-forward, report it and ask. If the browser cannot open, preserve the remote branch and provide the direct PR-creation target and prefilled content. If submission fails, do not click repeatedly. Never delete the remote branch or modify an already-created PR in v1.

On success, report the PR number/title/link, base/head and remote branch, batch commands/files, `PR_CREATED`, and that the v1 workflow has ended.

## Bundled Resources

- Read [references/en-page-authoring-rules.md](references/en-page-authoring-rules.md) completely in Phase 2.
- Run [scripts/validate_batch.py](scripts/validate_batch.py) for repository inspection and hard validation. Use `--help` for its complete CLI.
