# githubflow--pixelbrand

## Branch promotion and versioning

The repository uses this promotion flow:

```text
feature/* -> dev -> release -> prod
```

Merged pull requests create one semantic version tag on the merged promotion
commit:

| Target branch | Automatic bump | Automatic synchronization |
| --- | --- | --- |
| `dev` | Patch (`v1.0.0` -> `v1.0.1`) | None |
| `release` | Minor and reset patch (`v1.0.1` -> `v1.1.0`) | Merge `release` back into `dev` |
| `prod` | Major by default (`v1.1.0` -> `v2.0.0`) | Merge `prod` into `release`, then `release` into `dev` |

Add the `release:minor` or `release:patch` label to a pull request targeting
`prod` to override the default major bump. The workflows discover the highest
semantic version across all repository tags, so every branch stays aligned
after a minor or major promotion. When no tag exists, versioning starts at
`v0.0.0`.

### Repository setup

1. Create a repository secret named `GH_PAT` using a token owned by the
   repository administrator with repository contents write access. This
   personal-account repository cannot configure user-specific bypass actors,
   so administrator enforcement is disabled on the promotion branches for
   the mergeback jobs.
2. `dev`, `release`, and `prod` are protected with one required pull-request
   approval; force pushes and branch deletions are disabled. Keep the
   workflows' `GH_PAT` checkout for mergeback jobs so direct synchronization
   pushes can update the protected branches.
3. Create the `release:minor` and `release:patch` labels.
4. Open pull requests for each promotion. Once a pull request is merged, the
   matching workflow bumps the version and performs the required mergebacks.

The mergeback pushes do not close pull requests, so they do not trigger another
version bump. This keeps one version tag per promotion and prevents version
drift between `prod`, `release`, and `dev`.
