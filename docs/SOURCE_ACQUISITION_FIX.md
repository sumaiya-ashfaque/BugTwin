# BT-FIX-001 — Pin BugHub source acquisition to a commit tree

## Symptom
`RuntimeError: Source branch changed during discovery; rerun to pin a consistent revision` while calling `acquire_bughub` in the real-data Colab run.

## Root cause / change
The previous implementation fetched the recursive tree using the mutable branch name, then separately resolved the branch commit and compared its root-tree SHA with the earlier response. Even if both responses are valid, branch movement between requests can trigger the error. Tree and commit identifiers are distinct and must not be interchanged.

The corrected implementation first resolves the branch to a Git commit, captures its root-tree SHA, then fetches the recursive tree by that immutable tree SHA. It verifies the returned tree SHA and refuses truncated or malformed responses. Dataset downloads remain pinned to the resolved commit, and existing size/SHA-256 audits remain in place.

## Verification
- `test_source_resolves_pinned_commit_tree` checks immutable tree retrieval and distinct commit/tree SHAs.
- `test_source_rejects_mismatched_pinned_tree` checks that an inconsistent tree response is rejected.
- Existing M1/M2/M3 regression suite remains unchanged.

## Known limits
- Local fixture execution exercises the source-resolution logic using mocked responses; it does not validate live GitHub API access or the Eclipse corpus in Colab.
- If source acquisition progresses to a new data-format or duplicate-link error, diagnose that independently; do not bypass integrity gates.

## Release workflow
Apply on `fix/bughub-source-snapshot`, validate GitHub Actions and the real-data Colab run, then merge via a normal pull request.
