# Domain Playbook: Kubernetes

Load when the change touches controllers, operators, CRDs, or cluster config.

- **Controllers/reconciliation loops** — must be level-triggered (converge
  from any observed state), not edge-triggered (assume a specific prior
  event was seen). Confirm the loop re-reads current state rather than
  trusting its own cache or a status field it wrote earlier in the same run.
- **Eventual consistency** — the API server, informers, and etcd are not
  instantly consistent. Confirm the change doesn't assume a write is
  immediately visible to a subsequent read in another controller/informer.
- **CRDs** — check `status` vs `spec` ownership: only the controller should
  write `status`; only the user/upstream system should write `spec`. A
  webhook or another controller mutating `spec` is an ownership leak.
- **Finalizers** — if added, confirm they're removed on every exit path
  (including error paths), or the object is stuck on delete forever.
- **Optimistic concurrency** — resourceVersion conflicts are expected under
  concurrent updates. Confirm the code retries on conflict rather than
  treating it as a terminal error or silently overwriting.
- **RBAC** — new controller permissions should be scoped to the namespace/
  resource it actually needs, not cluster-wide by default.
- **Secrets in cluster state** — confirm secrets aren't leaking into
  `status`, events, or logs (`kubectl describe`/`get events` are widely
  readable).
