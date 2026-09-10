# HeatShift

HeatShift is a human-governed heat-safety operations platform for large outdoor workforces. It turns measured WBGT observations, task intensity, operational fitness flags, rest capacity and programme constraints into a safe proposed schedule. An independent rules engine validates the plan before a supervisor may approve it. Approval triggers multilingual worker notifications and a tamper-evident Decision Certificate.

This repository contains the working product surface, durable workflow API, deterministic safety engine, real-world evidence pack, AWS Innovation Sandbox control plane and tests.

## Five-minute winning flow

1. Open **Replay lab** and run the preserved 31 October 2025 record-heat event.
2. Watch the agent move through Sense → Check → Assess → Plan → Validate.
3. Review the exact schedule changes and six mandatory checks.
4. Approve as the safety supervisor. The plan cannot activate before this click.
5. Open **Notifications**, change language, and acknowledge the break as a worker.
6. Open **Evidence** and download the Decision Certificate.

## What is real

- The replay contains 96 quarter-hour periods, 15 NEA stations and 1,440 station-time rows.
- The 35.0°C WBGT peak is the preserved Sentosa Palawan Green S142 observation at 12:15 SGT on 31 October 2025.
- The ruleset is versioned from Singapore MOM heat-stress guidance.
- Hashes in `data/real-world/sha256_manifest.csv` verify the copied raw and normalised evidence.
- The live-map geometry is derived from the Elections Department's Electoral Boundary 2020 GeoJSON on data.gov.sg under the Singapore Open Data Licence.

Worker rosters, tasks, schedules and operational fitness flags are intentionally synthetic. Public worker or medical data is neither required nor appropriate for this demonstration.

## System boundary

```text
NEA observation → quality gate → deterministic risk rules → planning agent
       → independent validator → supervisor approval → worker notification
       → acknowledgement/help loop → Decision Certificate
```

The model can interpret operational context and explain an already validated result. It cannot set thresholds, waive constraints, approve a plan, write an active schedule or send a notification. Those controls remain in deterministic code and authenticated human actions.

## Application architecture

- `app/` — responsive operations interface and Cloudflare Worker-compatible route handlers.
- `lib/safety-engine.ts` — deterministic Singapore rules, planning and validation.
- `db/` and `drizzle/` — D1 schema and immutable generated migration for agent runs, decisions, notifications and hash-chained audit events.
- `data/real-world/` — raw observations, normalised records, provenance and integrity manifest.
- `services/aws_agent/` — Python Lambda control plane with optional Amazon Bedrock explanation through the Converse API.
- `infra/aws/` — low-cost AWS SAM stack: authenticated HTTP API, Lambda, DynamoDB, SNS, Cognito and bounded logs/concurrency.
- `SECURITY.md` — trust boundaries, threats and implemented controls.

## Local development

Requirements: Node.js 22.13+, pnpm 11+, and Python 3.12+.

```bash
pnpm install
pnpm run dev
```

Quality gates:

```bash
pnpm run typecheck
pnpm run lint
python scripts/verify_real_data.py
python services/aws_agent/test_app.py
pnpm run build
```

## AWS Innovation Sandbox deployment

Do not paste AWS credentials into chat, source files or Git. Lease the sandbox account, configure its temporary credentials locally, and keep the deployment in `us-east-1`.

```powershell
.\scripts\deploy-sandbox.ps1
```

The default deployment leaves Bedrock disabled, allowing the deterministic control plane to be tested first. Enable the model explanation only after the selected model is available:

```powershell
.\scripts\deploy-sandbox.ps1 -EnableBedrock
```

The stack uses on-demand DynamoDB, ARM Lambda, seven-day logs, reserved concurrency and API throttles to stay suitable for a short-lived sandbox. Review the CloudFormation change set before deployment and delete non-retained demo resources after judging. The operations table is intentionally retained to avoid accidental evidence loss.

## Free public Cloudflare deployment

The full-stack web application can be published on Cloudflare Workers without exposing an API key in the browser. This follows Cloudflare's current recommendation for Vinext/Next.js applications and ships the server routes, D1 workflow database and static interface together.

The judge-facing interactive demo is live at `https://heatshift.pages.dev`. It intentionally keeps approval simulation in the browser, so public visitors cannot write arbitrary records to the production database. Rebuild and publish that shareable demo from PowerShell with:

```powershell
npm.cmd run cloudflare:pages:deploy
```

One-time setup:

```powershell
npm.cmd exec wrangler login
npm.cmd exec wrangler d1 create heatshift-production
```

Copy the returned D1 database ID into `wrangler.worker.jsonc`, then initialize and deploy:

```powershell
npm.cmd exec wrangler d1 migrations apply heatshift-production --remote --config wrangler.worker.jsonc
npm.cmd run cloudflare:deploy
```

Cloudflare authentication remains in Wrangler's local credential store; never add API tokens to this repository.

## Data and safety disclaimer

HeatShift is decision support, not a medical device and not a substitute for a competent person, local risk assessment, emergency procedures or current regulatory advice. The source pack records access dates and limitations; refresh and re-validate the rules before any real deployment.
