# HeatShift security model

## Invariants

1. A model output is untrusted input.
2. Measured WBGT and the versioned ruleset are authoritative.
3. A schedule is inert until deterministic validation passes.
4. Only an authenticated supervisor action may activate a plan.
5. Worker notifications are derived from the approved record, never from free-form model text.
6. Every transition is attributable and exportable.

## Trust boundaries and controls

| Risk | Control implemented |
| --- | --- |
| Prompt injection changes a threshold | Thresholds live in `lib/safety-engine.ts` and `services/aws_agent/app.py`; the model only explains a validated plan. |
| Fabricated or stale environmental data | Source, station, timestamp and value stay together; real files are checked against SHA-256 hashes; the UI labels replay versus live state. |
| Client bypasses approval | `/api/decisions` re-loads the stored run and re-validates it server-side before an atomic decision write. |
| Duplicate clicks or replayed requests | Decision writes require an idempotency key and the database has a unique index on `agent_run_id`. AWS state transitions use conditional updates. |
| Unauthenticated data access | Sites write routes read trusted authenticated-user headers server-side; the AWS HTTP API uses a Cognito JWT authorizer. |
| Cross-site calls | AWS CORS is restricted to the deployed HeatShift origin. No wildcard origin is used. |
| Excessive permissions | AWS Lambda has only table CRUD, topic publish and the selected Bedrock model invocation. Credentials never enter browser code. |
| Sensitive worker health data leaks | Only pseudonymous IDs and minimum operational flags are model-visible. Diagnoses, medication and other medical details are excluded. |
| Notification ambiguity | Messages contain action, start time, duration and rest location, plus Received / Need help / Cannot comply responses. |
| Evidence is edited after the fact | Decision payloads receive SHA-256 digests and append-only audit-event hashes. |
| Cost or denial-of-service spike | API throttles, five reserved Lambda executions, small request limits, short timeouts and on-demand storage bound the demo. |
| Dependency compromise | Versions are locked; pnpm build scripts are allowlisted only for `esbuild`, `sharp` and `workerd`. |

## Data retention

The hosted prototype stores decisions and acknowledgements. Production deployment must add an organisation-approved retention schedule, tenant key management, incident response, backups and deletion workflows. The AWS demo table uses `Retain` to prevent accidental loss during a short-lived CloudFormation teardown; delete it explicitly when evidence retention is no longer required.

## Secrets

- Never commit `.env` files, AWS access keys, session tokens or Sites source credentials.
- Use temporary Innovation Sandbox credentials only in the local AWS credential provider chain.
- Use hosted secret/environment controls for runtime values.
- Rotate or revoke any credential that appears in logs, messages or screenshots.

## Responsible disclosure

Do not use this hackathon prototype for live workforce safety decisions. Report security issues privately to the project team with the affected route, reproduction steps and expected impact.
