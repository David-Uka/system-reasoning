# Domain Playbook: CI/CD

Load when the change touches pipeline config, build jobs, or deploy workflows.

- **Artifact flow** — confirm the artifact deployed is the same one that was
  built and tested (by digest/hash), not rebuilt at deploy time from
  possibly-different source.
- **Cache invalidation** — a cache key that's too broad silently serves
  stale dependencies/build output; too narrow defeats the cache. Confirm
  the key includes everything that should invalidate it (lockfile hash,
  compiler version, etc.).
- **Concurrency groups** — `cancel-in-progress` on a deploy job can cancel a
  deploy mid-apply, leaving infrastructure in a partial state. Confirm
  cancellation is safe for every job it's applied to, not copy-pasted from a
  build job where it was safe.
- **Rollback** — confirm there's an actual rollback path (previous artifact
  redeployable, migration reversible) and it's been exercised, not just
  assumed to exist because "we can just redeploy the old tag."
- **Secrets** — confirm secrets used in a job are scoped to the environment/
  branch that should have them, and forked-repo PRs can't exfiltrate them
  via `pull_request_target` + untrusted code checkout.
- **Approval gates** — confirm a required manual approval can't be
  bypassed by a differently-named workflow path that reaches the same
  deploy target.
