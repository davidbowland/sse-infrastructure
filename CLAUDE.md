# Project Guidelines

**Always commit changes** after completing work unless explicitly told not to.

This is an **infrastructure repo**: an AWS SAM/CloudFormation template, a
deploy script, and DNS/role-assumption helpers. There is no application code
here — the Lambda source for the API lives in the sibling `sse-api` repo, and
the static site source lives in the sibling `sse-ui` repo. Keep changes
declarative and reviewable.

## Layout

- `template.yaml` — the SAM/CloudFormation stack (primary artifact): deploy
  bucket (`LambdaBucket`), the `CloudFormationRole` and `PipelineRole` used by
  CI, the `SseApiUser`/`UiUser` IAM users that hold the GitHub Actions static
  keys, and a Cognito `IdentityPool`/`UnauthenticatedRole`.
- `deploy.sh` — assumes the scoped `developer` role (via
  `scripts/assumeDeveloperRole.sh`) and runs `sam deploy` against the test
  stack. CI deploys via `.github/workflows/pipeline.yaml`.
- No `src/`, no jest, no eslint. `lint` is `prettier --write .` only.

## Commands

- **Format:** `npm run lint` (prettier only).
- **Validate template:** `sam validate --lint`
- **Deploy (local/manual):** `npm run deploy` (wraps `./deploy.sh`) — assumes
  the scoped `developer` role, then `sam deploy` to the **test** stack. CI
  handles both test and production deploys on push.
- Never run a deploy that targets production without an explicit request.

## Style

- Prefer **functional, declarative** template composition; avoid copy-paste
  between the testing and production parameter sets — the template
  parameterizes with `Environment` (`prod`/`test`) via `Mappings.EnvironmentMap`.
- Keep resource logical IDs and parameter names consistent with the sibling
  `*-infrastructure` repos' conventions (clones share shape — drift is entropy,
  not intent).

## Security (CloudFormation)

- **No secrets in plaintext CFN parameters.** Where a parameter carries an API
  key or token, it MUST have `NoEcho: true`.
- **IAM least privilege:** scope actions to specific resource ARNs where
  practical. `CloudFormationRole` is intentionally broad by construction (a CF
  execution role needs to create/modify most resource types) but explicitly
  denies `iam:*User`, `organizations:*`, and `account:*`.
- Known, already-flagged issues that are **out of scope for routine changes**
  (separate passes; do not "fix" these incidentally while making an unrelated
  edit):
  - The CI pipeline currently assumes `arn:...:role/full-access` rather than
    this template's own scoped `sse-pipeline*` roles.
  - `SseApiUser`/`UiUser` hold long-lived static access keys for GitHub
    Actions (no OIDC yet).
  - The `IdentityPool`/`UnauthenticatedRole` (`AllowUnauthenticatedIdentities:
true`, grants `transcribe:StartStreamTranscriptionWebSocket`) is a vestige
    of a retired Amplify/Transcribe voice feature in `sse-ui` — the UI no
    longer imports Amplify, so this pool is currently unused live infra with
    an anonymous-access grant. Removal must happen in lockstep with confirming
    nothing else depends on it.
  - Two SSM parameters used by `sse-api` (`/sse/recaptcha-secret-key`,
    `/sse/suggest-claims-url`) are created out-of-band and are not managed by
    this template.
- **Log groups** (defined in `sse-api`, not here): `RetentionInDays: 30` with
  an ERROR subscription to `log-subscriber` — keep that pattern if adding new
  log-producing resources to this stack.

## Hygiene

- `.gitignore` covers `.aws-sam/`, `*.log`, `.DS_Store`.
