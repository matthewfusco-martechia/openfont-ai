# Plause

Plause is a cloud personal AI agent designed to live in Apple Messages. The Messages conversation is the primary experience. The web app provides sign-in, account controls, channel linking and staff support, with authenticated web chat available while Apple approval is pending.

The code includes Google sign-in, verified channel linking, a Sunshine Conversations Apple adapter, durable conversations and delivery, saved memory, reminders, exact action approvals, optional research and Google Calendar, staff ownership and replies, privacy erasure, backup/restore verification and isolated end-to-end tests. An isolated live OpenAI smoke passed three cases. All four local Google identity checks passed: initial OAuth/login/session, sign-out, returning sign-in with signup closed, and fresh confirmation. **Separate cloud identity, Calendar, Apple, search and deployed-service acceptance remain outstanding.** The primary preview has one real Google account alongside three fictional invitation accounts; model/messaging remain mock and Apple is off.

Opt-in Messages onboarding gives an unlinked sender a deterministic AI welcome and sign-in URL, then distinct proof-received and linked notices. It creates no private conversation or model turn before account confirmation. [Onboarding design](docs/messages-onboarding.md) records the delivery and privacy boundaries.

## Run locally

Use Node.js 24.10 or newer.

```sh
npm ci
npm run setup
npm run dev
```

Open [Plause locally](http://localhost:8787). Local invitation keys are in `.local/invitations.json`; open that file locally and use an account's `inviteToken`. Do not paste keys in chat or commit them. Setup preserves existing keys/accounts and applies additive migrations. A running local server wakes scheduled recovery every 15 seconds; D1 owns the durable state.

Keep `.dev.vars`, `.local`, `.wrangler` and backups private. Preserve the matching encryption key or complete keyring with every database backup. Local invitations and the operator bearer token work only in the local environment.

## Verify the complete implementation

```sh
npm run check
npm run test:e2e
npm run build
```

`check` runs TypeScript, Worker integration tests and Node CLI/security tests. `test:e2e` owns a temporary server, database, random port and fictional identities. It tests real HTTP authentication, signed ingress, channel proof/confirmation, tenant isolation, memory, reminders, approvals, staff replies, SIGKILL/restart recovery, export pagination, account erasure and an actual quiescent D1 backup/offline restore. It never reads local invitation/provider credentials or uses the preview database. Its report is `.local/e2e-report.json`. Provider API tests use controlled fixtures and deny real network calls.

Historical complete verification, before current UI/enrollment changes, passed TypeScript, 1,363 tests (1,259 Worker/104 Node), 14 HTTP E2E scenarios and build dry run. Its E2E/source hash matches the earlier live model report and is not a current-source rerun. The sign-out change passed 5 existing focused UI tests/syntax checks. Controlled cloud enrollment passed 95 focused Worker tests, 15 CLI tests and TypeScript, with no blockers in read-only review. The first broad Worker rerun passed 1,283/1,284; one cloud-delivery fixture inherited local Google configuration and is being isolated/retested before a full rerun. Do not treat the current full suite as green. See [the handoff](HANDOFF.md).

`build` is a deployment dry run. It publishes nothing. CI runs the same checks and uploads the compact E2E report; no remote CI result is claimed before a repository/run exists. The four local Google identity interactions are verified by user confirmation and count-only checks. Broader browser visual, interaction and accessibility acceptance remains unverified; the earlier automated-browser URL restriction and API/static checks do not establish those gates.

With the preview running, `npm run demo`, `npm run demo:memory` and `npm run demo:agent` exercise additional fictional mock flows.

## Agent behavior

- Conversation work, provider events, logical messages, model attempts and the outbox are durable and independently deduplicated. Ownership, account context, channel binding, leases, consent and maintenance gates are rechecked before consequential work.
- Hosted AI uses a bounded OpenAI Responses adapter, exact input-token preflight, conservative per-user/service spending reservations, encrypted result recovery and one paid generation attempt per accepted input. Unknown charges remain held.
- Saved facts require explicit commands. Send `memory help`; up to 10 facts of 300 UTF-16 units are supported. Ordinary chat and model output do not save facts. Forget/edit/off/on reset future AI context and cancel stale work; existing transcripts remain until account erasure.
- Once/daily/weekly reminders respect IANA time zones, DST, quiet hours, notification preferences and versioned occurrences. Proactive Apple delivery additionally requires channel approval, an explicit deployment policy flag and separate user consent.
- Model tools can propose reminders or calendar events and request a bounded calendar read. Writes require a separate exact expiring one-use approval; a bare yes does nothing. Send `action help` or `google calendar help`. Simulated calendars remain clearly labeled and are disabled for the production transport.
- Research requires an explicit public query such as `search: public libraries in Boston`. No private facts or chat history are forwarded as search arguments. Search and Google Calendar are disabled until separately configured.
- A human support request pauses model replies. Authorized staff use fresh verified sessions, roles and expiring conversation claims. Replies are durably queued and revoked claims cannot send. Provider-owned or unknown-control conversations require provider-side resolution before Plause staff can resume them.
- Web messages become delivered only after an authenticated display receipt. Sunshine API acceptance is recorded as sent. Unknown provider sends use read-only reconciliation and are never blindly posted again.

## Cloud configuration and live acceptance

The confirmed domain is **plause.si**, with `https://staging.plause.si` proposed for staging. Public DNS now delegates to Cloudflare; application hosting and deployment remain pending. Run `npm run readiness` for a read-only, secret-free report of missing configuration, then follow [account setup and real-device acceptance](docs/live-acceptance.md). Exit 2 means live configuration is incomplete; the local mock can still be configured.

After explicit user approval, exactly four empty staging resources were created: `plause-staging-db`, `plause-staging-jobs`, `plause-staging-dlq` and private `plause-staging-privacy-ledger`. Independent info checks matched D1 identity with 0 tables, found no queue producers/consumers and zero observed backlog, and checked the new private Standard/ENAM bucket returned 0 objects. Its list response lacked an explicit truncation flag; the observation is not an exhaustive-list guarantee. Actual IDs and verification evidence stay private in the 0600 plan, state `provisioned_verified`. Local configuration/bootstrap hashes were preserved. No remote migrations, secrets, Worker/domain deployment, billing changes or live Messages occurred.

The imported OpenAI key was used successfully in a fresh isolated live evaluation on 2026-11-02: three completed `gpt-6-luna` generations, 3/3 cases passed, zero unknown charge holds and verified temporary process/data cleanup. The app ledger estimated US$0.001433; that is not an invoice amount. That evaluation preserved the primary preview's then-current encryption/settings. The primary model remains mock (`MODEL_PROVIDER=mock`, `LIVE_MODEL_ENABLED=false`). The user reports billing ready and API credits were verified privately; no purchase was made by the agent.

The Plause development Google project and local identity client are configured. User confirmation and count-only checks verified initial login, sign-out, returning to the same account with signup closed, and fresh confirmation. Final state: 1 Google identity, 1 valid/fresh session and 4 successful login audits; the 3 fictional invitation accounts have 0 identity associations. Google identity remains enabled and signup false. No local manual identity check remains pending; the private 0600 count-only report is `.local/identity-acceptance.json` (passed=true). Encryption material and mock model/messaging were preserved. Calendar-specific credentials/grants and Sunshine credentials remain absent. The separate staging Google client download is pending. A new independent staging keyring, stored OpenAI key and one verified subject are prepared in a private pending secret file; signup stays off, with no operator grants or external escrow verification. Controlled exact-subject cloud enrollment is implemented and focused tests pass; the full regression rerun is still pending. Continue [staging preparation](docs/staging-proposal.md). `npm run configure:identity` makes no Google request; actual acceptance used the separate browser OAuth flow.

Start with [cloud deployment](docs/cloud-deployment.md), [identity and support](docs/identity-and-support.md), [Sunshine messaging](docs/sunshine-messaging.md), [operations](docs/operations-runbook.md), [privacy and restore](docs/privacy-and-restore.md), and [encryption rotation](docs/key-rotation.md).

`npm run configure:cloud -- <options>` creates a reviewed staging/production configuration from explicit resource IDs and protected secret files. It does not provision, deploy or send messages. Environments use separate D1 databases, queues/DLQs, privacy buckets, origins and private credentials. Production rejects mock inference/messaging. Authenticated web is the initial fallback; Apple requires actual approved brand/MSP configuration and staffed support arrangements.

`npm run configure:model -- --store-only` stores a key through a hidden prompt without enabling hosted requests. The protected-file form is `npm run configure:model -- --store-only --key-file PATH`; never put a key value in an argument. The current key has already been stored.

To explicitly run the isolated paid evaluation:

```sh
npm run eval:model -- --live
```

It uses a fresh temporary D1 database, fictional account, encryption keys and random port, reusing only the stored API key. It permits at most three serial generations with a US$0.03 app-ledger estimate limit, which is not a provider invoice cap. A private `0600` metadata report, including unknown charge holds, is retained under `.local/model-evaluations` before cleanup. The primary preview can remain mock; enabling or restarting it is unnecessary. The completed 3/3 run establishes the narrow hosted-model smoke result, rather than general assistant quality or Apple/device acceptance. Research has a separate protected-file setup.

Next are the staging OAuth client download, completed private secrets/config/dry run, full regression rerun and separately reviewed migrations/deployment. Controlled cloud enrollment requires a private 1–5 subject allowlist whenever cloud signup is enabled; it does not grant operator privileges. A fresh local 19-migration rehearsal passed with 46 tables and no users/identities/sessions; cloud D1 remains unmigrated. Apple/MSP approval, support staffing, provider/device acceptance, production restore/alerting and legal/retention review remain outstanding. Four empty staging resources exist; no real Apple send or Worker deployment occurred.

See [HANDOFF.md](HANDOFF.md) for the latest tested snapshot and remaining acceptance gates.
# openfont-ai
