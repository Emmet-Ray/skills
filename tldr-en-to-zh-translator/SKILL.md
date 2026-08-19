---
name: tldr-en-to-zh-translator
description: Standardize the end-to-end contribution workflow for adding missing Simplified Chinese translations to tldr-pages, including duplicate checks, translation, deterministic validation, human review, local commit, fork push, and browser PR handoff. Use when the user invokes this skill with one or more tldr command names and wants to add missing pages.zh translations in a local tldr-pages/tldr clone. Do not use for updating existing translations or authoring English pages.
---

# Translate tldr Pages to Simplified Chinese

Treat one skill invocation as one translation batch. Keep every user-provided command in the same branch, commit, and PR. Do not introduce a separate task-recognition stage or split the batch automatically.

## Scope

- Add only missing Simplified Chinese pages under `pages.zh`.
- Do not update, synchronize, or polish an existing Chinese page.
- End after the PR is created and verified.
- Do not track CI, handle review comments, monitor merging, or clean branches in v1.
- Run from the root of a local `tldr-pages/tldr` clone.

Use this state flow:

```text
REQUESTED
  -> READY_FOR_TRANSLATION
  -> TRANSLATED
  -> VALIDATED
  -> CONTENT_APPROVED
  -> LOCAL_FINALIZATION_APPROVED
  -> LOCALLY_FINALIZED
  -> REMOTE_PREPARATION_APPROVED
  -> PR_READY_FOR_REVIEW
  -> PR_CREATED
```

## Global Controls

- Treat the command set and resolved source/target mapping as the batch identity. If a command is added, removed, or mapped to another platform, return to preflight and rerun it completely.
- Aggregate all blockers found in a phase. Explain each cause, recommend a concrete resolution, and then ask the user what to do.
- Never silently drop a command, choose an ambiguous platform, stash work, overwrite a branch, bypass hooks, force-push, delete a branch, or rewrite commit history.
- Bind every approval to the exact files and plan shown to the user. Any page-content change invalidates `CONTENT_APPROVED`. Any material commit or remote-plan change invalidates its corresponding approval.
- Keep `CONTENT_APPROVED`, `LOCAL_FINALIZATION_APPROVED`, and `REMOTE_PREPARATION_APPROVED` distinct.
- Resolve paths relative to the repository root. Resolve bundled resources relative to this `SKILL.md`.

## Parse the User Input

- Accept only command names; do not require platform names, repository paths, or page paths.
- Accept one command or a comma-, Chinese-enumeration-comma-, newline-, or natural-language-separated list.
- Preserve spaces inside subcommands. Treat `git commit` and `git-commit` as equivalent lookup forms.
- If input boundaries are ambiguous, show the parsed command list, explain the ambiguity, recommend an interpretation, and ask the user to confirm within preflight.
- Preserve the user’s command order throughout the batch.

## Phase 1: Preflight

Enter with `REQUESTED`. Complete all checks before creating a branch.

1. Read the current `.github/PULL_REQUEST_TEMPLATE.md` and extract its new-page limit. The current expected limit is 5, but the repository document is authoritative.
2. Require a clean worktree using `git status --porcelain=v1 --untracked-files=all`. If dirty, list every change and recommend that the user save or finish it. Do not stash, commit, or delete it.
3. Inspect every remote URL. Identify `tldr-pages/tldr` by normalized URL, never by assuming the remote is named `upstream` or `origin`.
4. Run `gh auth status`. If unauthenticated, recommend `gh auth login` and pause. Never handle or print credentials.
5. Fetch the official upstream `main` explicitly.
6. Run the bundled resolver against the fetched ref:

   ```text
   python <skill-dir>/scripts/validate_batch.py resolve \
     --repo <repo-root> \
     --ref <official-remote>/main \
     --json \
     <commands...>
   ```

7. Require exactly one English source page and no Chinese target page for every command.
8. Query open upstream PRs with file data, using a sufficiently high limit:

   ```text
   gh pr list --repo tldr-pages/tldr --state open --limit 1000 \
     --json number,title,url,files
   ```

   Compare actual changed file paths with every target page. Do not infer duplicates from PR titles alone.
9. Create one branch from the fetched upstream `main` only after every check passes.

Name the branch `translate-zh/<batch-slug>`:

- Normalize a subcommand to a hyphenated page name, such as `git commit` -> `git-commit`.
- Join multiple commands with `_`, such as `translate-zh/jq_git-commit_git-add`.
- Do not add a date, hash, or random suffix preemptively.
- If the local name already exists, do not reuse or delete it. Recommend a readable numeric suffix and ask the user.

Report the source/target mapping, all check results, the branch name, and `READY_FOR_TRANSLATION`.

## Phase 2: Translate the Whole Batch

Enter only with `READY_FOR_TRANSLATION`.

Read [references/zh-translation-rules.md](references/zh-translation-rules.md) completely before translating. Then read every fixed and conditional repository reference routed by that document. Current repository rules outrank the bundled summary.

Apply these non-negotiable boundaries:

- Before drafting each page, search `pages.zh` for existing pages in the same command family and inspect the closest sibling commands. Reuse established translations for shared terminology, recurring descriptions, and placeholders when they fit the English source. For example, if `docker compose stop` and `docker compose down` are missing but `docker compose up` exists, use the latter as a wording reference.
- Mirror each current English page’s structure, meaning, example count, and example order.
- Do not add, remove, merge, split, or reorder examples.
- Translate natural-language descriptions, standard phrases, and translatable placeholders.
- Preserve command names, subcommands, options, URLs, syntax, fixed literals, and behavior.
- For an alias page, use the repository’s `scripts/set-alias-page.py` with locale `zh`, then verify the result against the current template.
- Finish every page in the batch before moving to validation.

Treat sibling pages as wording references only: the current English source remains authoritative for content, structure, and command behavior. For uncertain terminology or multiple reasonable expressions, use repository context and existing related `pages.zh` pages to choose the best draft. Record the term, chosen expression, alternatives, and reason. Do not repeatedly interrupt the user. Pause only if the English meaning cannot be understood reliably.

Report the created drafts, uncertainty list, and `TRANSLATED`.

## Phase 3: Run Hard Validation

Enter only with `TRANSLATED`. Do not judge naturalness, translation preference, or other human-language quality here.

Run all of the following against the exact target paths:

1. Run the project-local Markdown linter.
2. Read the current `scripts/test.sh` and run the project-local `tldr-lint` with the exact ignore codes currently used for `pages.zh`. Do not use a bare invocation whose behavior differs from CI.
3. Run the bundled validator with every source/target pair and scope checking enabled:

   ```text
   python <skill-dir>/scripts/validate_batch.py validate \
     --repo <repo-root> \
     --check-scope \
     --json \
     --pair pages/<platform>/<command>.md=pages.zh/<platform>/<command>.md \
     [--pair ...]
   ```

The bundled validator checks mapping, titles, structural order, header/example/command counts, URLs, inline command references, command integrity with translatable placeholders masked, trailing whitespace, final newlines, and worktree scope. It is read-only.

Automatically fix only deterministic hard failures. After any fix, rerun the entire phase. If a failure cannot be fixed reliably, aggregate it, recommend a resolution, and ask the user. Do not advance while any hard failure remains.

Report the hard-check summary, automatic fixes, carried uncertainty list, and `VALIDATED`.

## Phase 4: Human Review and Iteration

Enter only with `VALIDATED`. By default, report exactly:

- Batch commands and new-file list.
- Recorded uncertain terms and expressions.
- Hard-validation summary.
- An explicit statement that nothing has been committed, pushed, or submitted as a PR.

Do not attach full pages, a complete diff, or auto-fix details unless requested. Ask the user to inspect the local pages.

For every requested change:

1. Edit the affected page.
2. Invalidate the prior `VALIDATED` result.
3. Rerun the entire hard-validation phase.
4. Present the same four-item review package again.

Do not require the user to respond to every uncertainty individually. An explicit overall content approval after disclosure accepts the current wording. Never infer approval from silence or an ambiguous reply.

On explicit approval, report `CONTENT_APPROVED`. Do not commit yet.

## Phase 5: Local Finalization

Enter only with `CONTENT_APPROVED`.

1. Read the current `CONTRIBUTING.md` section “Commit message and PR title.”
2. Draft one commit message for the entire batch. Use current repository rules and recent accepted translation commits only to resolve uncovered ambiguity.
3. Show the exact target files and proposed commit message. Ask for explicit local-finalization authorization.
4. After approval, report `LOCAL_FINALIZATION_APPROVED` and rerun the entire hard-validation phase.
5. If any file changes during final validation, invalidate content approval and return to Phase 3 and human review.
6. Confirm that the worktree contains only the expected new target files.
7. Stage exact paths with `git add -- <target-pages...>`; never use a broad add.
8. Run `git diff --cached --check`, inspect `git diff --cached --name-status`, and inspect the staged content.
9. Create exactly one commit with the approved message. Do not use `--no-verify`.
10. Verify the new commit’s hash, message, files, content, and post-commit worktree status.

If a hook or commit fails, report the original error and staged state. Do not reset, delete files, bypass the hook, or rewrite history. If a successful commit is later found wrong, stop and ask the user; do not amend or reset automatically.

On success, report only the commit hash/message, committed files, final validation and worktree result, and an explicit statement that nothing has been pushed or submitted. Report `LOCALLY_FINALIZED`.

## Phase 6: Remote Preparation and PR Handoff

Enter only with `LOCALLY_FINALIZED`. Do not repeat preflight’s upstream fetch, page-existence check, or open-PR duplicate scan by default.

1. Use `gh api user` to identify the current account.
2. Reuse the official upstream found in preflight. Identify the user fork by URL, fork relationship, and permission with `gh repo view`; do not assume `origin` is the fork.
3. Require a unique writable fork remote. Never push to the official upstream.
4. Read the current commit/PR title rule and `.github/PULL_REQUEST_TEMPLATE.md`.
5. Default the PR title to the commit message. Preserve the complete current PR template.
6. Check only boxes supported by actual evidence. Check the human-review item only because the batch reached `CONTENT_APPROVED`. Do not invent command versions or issue references. Use `Closes` only when the PR fully resolves a known issue.
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

10. Stop at `PR_READY_FOR_REVIEW`. Ask the user to inspect the full diff, base/head, title, body, and checklist in the browser.
11. Let the user click “Create pull request” by default. If the user explicitly asks the agent to submit, reread the visible base/head, title, and body first and submit only if base/head still match the authorized plan.
12. After creation, use `gh pr view` or `gh pr list` to verify the PR number, title, URL, base/head, and state.

If push fails, do not force. If a remote branch already exists or the push is non-fast-forward, report it and ask. If the browser cannot open, preserve the remote branch and provide the direct PR-creation target and prefilled content. If submission fails, do not click repeatedly. Never delete the remote branch or modify an already-created PR in v1.

On success, report the PR number/title/link, base/head and remote branch, batch commands/files, `PR_CREATED`, and that the v1 workflow has ended.

## Bundled Resources

- Read [references/zh-translation-rules.md](references/zh-translation-rules.md) completely in Phase 2.
- Run [scripts/validate_batch.py](scripts/validate_batch.py) for source resolution and hard validation. Use `--help` for its complete CLI.
