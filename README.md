# oidc

Public OIDC discovery documents for the homelab Kubernetes cluster, served via
GitHub Pages so Tailscale Workload Identity Federation (WIF) can verify the
Tailscale k8s operator's projected ServiceAccount token.

**Issuer:** `https://jrichlen-lab.github.io/oidc`

Everything here is **public, non-secret metadata** — a discovery document and a
set of public signing keys. There is nothing sensitive to protect; the only
requirements are integrity (serve exactly the published bytes), availability,
and valid public TLS (GitHub Pages provides managed TLS automatically).

## Served paths

| Path | File |
|------|------|
| `/.well-known/openid-configuration` | `.well-known/openid-configuration` |
| `/openid/v1/jwks` | `openid/v1/jwks` |

`.nojekyll` is present so the Pages build serves the `.well-known/` dotfolder
verbatim instead of running Jekyll (which would drop dotfolders).

## JWKS is a placeholder until cutover

`openid/v1/jwks` currently contains `{"keys":[]}`. At cutover it must be
replaced **verbatim** with the cluster's real signing keys:

```sh
kubectl get --raw /openid/v1/jwks > openid/v1/jwks
```

Do not reformat or re-key the captured JWKS. Re-publish `/openid/v1/jwks`
whenever the cluster's SA signing keys rotate, or in-flight operator tokens
will fail validation.

## Consumers (must match the issuer byte-for-byte)

The issuer string `https://jrichlen-lab.github.io/oidc` is consumed, identically,
by:

- the `issuer` field of `.well-known/openid-configuration` (this repo),
- the kube-apiserver primary `--service-account-issuer`,
- the Tailscale federated-identity **Issuer URL** for the k8s operator,
- `SECOND_ISSUER` in `jrichlen-lab/tailnet-federation` `ci/reconcile.py`.

See `jrichlen-lab/homelab` `docs/oidc-hosting.md` and
`docs/operator-wif-cutover.md` for the full runbook.
