---
applyTo:
  - "**/Dockerfile"
  - "**/*.Dockerfile"
  - "**/*.tf"
  - "**/k8s/**/*.yaml"
  - "**/k8s/**/*.yml"
  - "**/helm/**/*.yaml"
  - "**/helm/**/*.yml"
---

# Infrastructure review standards

## Dockerfile

- `FROM` must pin to a digest, not a moving tag. Format: `image:tag@sha256:...`.
- Multi-stage build required for any compiled language (Go, Rust, Java, C#).
- Final stage must `USER` a non-root UID. Flag any image running as root.
- `COPY` over `ADD` unless extracting a tarball.
- `apt-get install` must use `--no-install-recommends` AND clean up
  `/var/lib/apt/lists/*` in the same `RUN` layer.
- `pip install`, `npm install`, `apk add` results must be cleaned (`--no-cache-dir`,
  `--no-cache`, `--no-cache`) to avoid bloat.
- `HEALTHCHECK` required for long-running services.
- No `curl ... | sh` from the network. Pin and verify.

## Kubernetes manifests

- Every container must declare `resources.requests` AND `resources.limits` for
  CPU and memory. Missing either is `[must-fix]`.
- Every Pod must define `livenessProbe` AND `readinessProbe`.
- No `image: *:latest` tags. Require explicit version or digest.
- `securityContext` should set:
  - `runAsNonRoot: true`
  - `allowPrivilegeEscalation: false`
  - `readOnlyRootFilesystem: true` (or explain why not)
  - `capabilities.drop: ["ALL"]`
- Never `hostNetwork: true`, `hostPID: true`, or `privileged: true` without an
  inline comment justifying it and linking to an approval issue.
- Secrets must come from `secretRef` or external secret managers, never `env` plaintext.
- `NetworkPolicy` should exist for any new namespace.

## Terraform

- No hardcoded credentials, account IDs, ARNs, or region literals. Use variables
  or `data` sources.
- Every taggable resource needs tags including at minimum: `Owner`,
  `Environment`, `CostCenter`, `ManagedBy = "terraform"`.
- S3 buckets must have `block_public_access` enabled and `versioning` on.
- RDS, Aurora, DynamoDB must have `storage_encrypted = true` (or KMS equivalent).
- Security groups must not allow `0.0.0.0/0` on management ports
  (22, 3389, 5432, 3306, 6379, 27017). Flag as `[must-fix]`.
- IAM policies wider than necessary (`*` action or `*` resource) → `[must-fix]`
  unless commented with justification.
- Module versions must be pinned with `version = "~> X.Y"` or stricter.
