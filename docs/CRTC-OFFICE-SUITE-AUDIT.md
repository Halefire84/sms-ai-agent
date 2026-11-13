# CRTC Office Suite / PhoneNode — Initial Audit and Build Charter

**Date:** 2026-11-13  
**Status:** AUDIT IN PROGRESS — no phone-side test or deployment performed  
**Budget:** $0 additional spend unless separately approved  
**Scope:** PhoneNode communications, CRTC receptionist, and bookkeeping preparation

## 1. Executive finding

The existing `Halefire84/sms-ai-agent` repository is the best starting point for the phone-side SMS adapter. It is MIT-licensed and already contains a Termux SMS handler, Python service structure, SQLite-oriented persistence, FastAPI UI, LLM adapters, rate limiting, response guardrails, and Termux setup instructions.

It is not yet a complete commercial receptionist or office suite. Its README explicitly limits it to SMS; it does not establish a production voice receptionist, web lead CRM, email workflow, appointment calendar integration, accounting ledger, receipt OCR pipeline, tenant isolation, consent ledger, or tested Android 24/7 reliability.

Do not describe it as production-ready for CRTC customers until the gaps below are implemented and independently tested.

## 2. Repositories inspected

### Existing project boundaries

- `Halefire84/sms-ai-agent` — SMS/Termux adapter candidate; MIT license verified in `LICENSE`.
- `Halefire84/starship-vault` — phone-first encrypted vault / Gatekeeper. Its README describes Android Keystore + XChaCha20-Poly1305, scoped agent delegation, and audit logging. Keep secrets and approvals here; do not duplicate the vault.
- `Halefire84/starship-vault-relay` — Nod Relay. README describes short-lived, phone-signed grants, Ed25519 verification, separate agent/device authentication roots, and a hash-chained audit. Preserve its trust boundary; do not make the relay a credential store.
- `Halefire84/one-app-phone` — native Android chat-first client. README says research is done and implementation has not started; it explicitly says no CRTC-hosted backend. Do not silently change that product boundary.
- `Halefire84/STARSHIP` — repository is described as a private source-history backup. It is not yet confirmed to be the active PhoneNode runtime repository.

### Historical device evidence (not a fresh test)

A 2026-11-03 master-plan entry records that the phone was reachable over Tailscale with key-only SSH, while the SMS agent was blocked because the installed Play Store Termux lacked Termux:API. It proposed installing Termux and Termux:API from F-Droid, then rechecking permissions. This is historical evidence only; current device state is unknown.

A 2026-11-01 bridge report says Termux items such as `~/sms-ai-agent`, `~/crtc-control-plane`, and `~/starship-claude-bundle` are on the phone and were not tested from the laptop in that report.

## 3. SMS AI Agent code review — initial pass

Files inspected: `README.md`, `LICENSE`, `requirements.txt`, `main.py`, `services/sms_handler.py`, `services/guardrails.py`, `core/security.py`, and `install.sh`.

### Reusable foundations

- MIT license permits modification and commercial reuse subject to retaining the copyright and license notice.
- Termux SMS adapter uses `termux-sms-send` and `termux-sms-list`.
- Python application structure separates core, LLM, rules, services, and UI.
- README describes SQLite message history, rate limits, echo/loop protection, PII/content guardrails, FastAPI UI, and Termux:Boot integration.
- Default web host in `main.py` is `127.0.0.1`, a safer local default than binding to all interfaces.

### Initial risks and unverified claims

- README's "production-ready" label is not supported by a fresh test run in this audit.
- README's "free model" path still depends on external internet and third-party model availability; it is not an offline guarantee.
- SMS send/read depends on Termux:API app, package, Android permissions, and carrier/SIM service.
- `main.py` accepts an API key as a command-line argument (`--api-key`). Do not use that path for real secrets until reviewed; process arguments can be exposed to local process inspection and shell history.
- `install.sh` performs package update/upgrade operations. Do not run it against the owner's phone as part of a read-only audit.
- The SMS handler and security modules need full review for shell invocation safety, message deduplication, idempotency, webhook validation, log redaction, key file permissions, and database encryption.
- SMS auto-reply defaults and rate limits need business-grade consent, STOP/HELP handling, quiet hours, human escalation, per-tenant isolation, and a hard global send kill switch before any commercial use.
- Voice calls, call recording, WhatsApp/Signal/other private messaging ingestion, and accounting integrations are not established by this repo's README.

## 4. PhoneNode communications architecture

Use a transport adapter model; do not pretend one radio or API can provide free nationwide cellular service.

Priority order:

1. **USB + ADB** — local diagnostics, controlled recovery, and laptop-to-phone administration while physically connected.
2. **SSH over Tailscale** — remote administration over an authenticated private network; key-only access, no public SSH port.
3. **Local Wi-Fi / Wi-Fi Direct** — device-to-device messaging when nearby; test hardware and Android API support on the actual Moto phone.
4. **Bluetooth / Nearby Connections** — short-range offline data and control channel; prototype only after device compatibility is proven.
5. **SIM SMS and voice** — optional carrier transport, subject to plan, coverage, Android permissions, carrier terms, and applicable law.

The phone is an edge communications and approval node, not a literal Ethernet switch or guaranteed always-on server. Ubuntu remains primary compute, database and AI execution. Phone services must tolerate Doze, battery restrictions, restarts, loss of network, and thermal limits. Android foreground-service and background-start rules must be respected.

## 5. Receptionist product scope

Build one cohesive CRTC Office Suite with modular capabilities:

### Receptionist
- Website inquiry and chat intake.
- Email intake and response drafting.
- SMS inbound/outbound only where device and policy permissions permit.
- Voice receptionist via a lawful telephony/VoIP provider or user-initiated call flow; no claim of free PSTN calling.
- Business hours, holiday schedule, FAQs, service catalog, location/service-area rules.
- Customer name, callback number, request, consent, preferred contact channel, urgency.
- Appointment availability and booking with conflict checks.
- Human handoff, callback queue, missed-message recovery, and conversation transcript.
- Clear disclosure when a customer is interacting with an automated assistant.

### Bookkeeping assistant
- Receipt and invoice upload, image/PDF parsing and OCR.
- Vendor/customer records and duplicate detection.
- Suggested categories, tax tags and confidence scores.
- Draft expense/income transactions with source-document references.
- Bank-feed import only through an explicitly authorized provider/connector; no credential scraping.
- Reconciliation suggestions and exception queue.
- Monthly summaries and accountant export (CSV/standard accounting import format).
- Human review before posting, paying, refunding, filing, or changing bank details.

### Business operations
- Lead/customer CRM with tenant separation.
- Appointment and follow-up reminders.
- Daily owner digest and unresolved-item queue.
- Audit trail of every automated action, data source, model, policy decision, and human approval.
- Export, retention and deletion workflows.

## 6. Safety, privacy, and compliance gates

- No SIM cloning, carrier authentication bypass, unauthorized network access, spoofing, jamming, or interception of third-party communications.
- No covert access to other messaging apps. Use documented APIs and explicit user/customer authorization.
- SMS and call-log Android permissions are restricted by Google Play policy; apps generally need to be the active default SMS/Phone/Assistant handler for the relevant restricted permissions, subject to review and exceptions. Termux sideload behavior does not remove legal, platform, carrier, or privacy obligations.
- For marketing robotexts/robocalls, implement documented consent capture and proof, sender identification, opt-out processing, suppression lists, quiet-hour controls, and audit history. Informational and transactional messages must remain within the consent and purpose actually given.
- Email campaigns must implement accurate sender information, non-deceptive subject lines, a valid postal address, and working opt-out handling where CAN-SPAM applies.
- Collect the minimum customer data needed. Encrypt sensitive data at rest and in transit; enforce per-business tenant boundaries; never place secrets or full payment credentials in model prompts or logs.
- No autonomous money movement, tax filing, bank-account changes, or irreversible ledger posting in v1.

## 7. Required acceptance tests

### Phone / communications
- Device identity and Android version captured without exposing personal secrets.
- Termux and Termux:API package/app versions and permission state captured.
- ADB trusted-host list and USB-debugging state captured.
- SSH is key-only and reachable only over approved local/Tailscale paths.
- Reboot recovery and background survival measured; no assumed 24/7 uptime.
- Outbound test message only to an owner-controlled test number, after explicit test approval.
- Inbound test message deduplicated; STOP suppression and kill switch verified.
- Network outage and relay outage leave durable queued messages with no duplicate sends.

### Receptionist
- Web/email inquiry stored once with unique event ID.
- Duplicate webhook delivery does not create duplicate customer or appointment records.
- Consent source and purpose retained with timestamp.
- Unknown or risky requests escalate to a human.
- Calendar conflicts are rejected; timezone and business hours tested.
- AI outage produces a truthful callback acknowledgement, not a fabricated answer.

### Bookkeeping
- OCR result linked to original document hash.
- Duplicate invoice/receipt detection.
- Amount, currency, date, tax and vendor confidence thresholds.
- Low-confidence items routed to review.
- No posting or payment side effect without explicit authorization.
- Export totals reconcile against the prepared entries.

## 8. Work sequence

1. Complete repository and dependency/license/security audit; preserve current branches and history.
2. Obtain a read-only Termux/Android inventory from the actual Moto phone; do not reinstall, upgrade, debloat, wipe, or alter permissions during this pass.
3. Review Knoxx/Gatekeeper and Nod Relay contracts and failure chronology; preserve logs and last-known-good versions.
4. Harden and test SMS adapter behind disabled-by-default outbound sending.
5. Implement durable local-first transport queue and USB/Tailscale/Wi-Fi Direct capability probes.
6. Build receptionist intake, CRM, scheduling, and human handoff.
7. Build receipt/invoice OCR and bookkeeping preparation.
8. Integrate the existing vault/relay for approval; no credentials in agent context.
9. Run end-to-end tests with synthetic customer data and owner-controlled phone numbers.
10. Package, price-test, and deploy only after privacy, compliance, security, reliability, and unit-economics gates pass.

## 9. Current evidence status

| Item | Status |
|---|---|
| SMS AI Agent repository and MIT license | PASS — repository files inspected |
| SMS/Termux source structure | PASS — initial source review |
| Active PhoneNode runtime repository identified | NOT TESTED |
| Current Moto phone inventory | BLOCKED — needs read-only device output |
| Current Termux:API permission state | BLOCKED — needs device output |
| Current SSH/Tailscale connectivity | NOT TESTED today |
| Knoxx/NOD Relay end-to-end health | NOT TESTED |
| SMS send/receive on physical device | NOT TESTED |
| Voice receptionist | NOT BUILT |
| Customer-facing receptionist package | NOT BUILT |
| Bookkeeping/OCR workflow | NOT BUILT |
| Commercial compliance/security acceptance | NOT TESTED |

## 10. Read-only phone inventory request

Run from Termux only after confirming you are in the intended Termux session. This command prints versions and service state, not credentials or message contents:

```bash
printf '\n== DEVICE ==\n'
getprop ro.product.manufacturer
getprop ro.product.model
getprop ro.build.version.release
getprop ro.build.version.sdk
printf '\n== TERMUX ==\n'
printf 'TERMUX_VERSION=%s\n' "$TERMUX_VERSION"
command -v sshd || true
command -v termux-sms-list || true
command -v termux-sms-send || true
command -v termux-battery-status || true
printf '\n== PACKAGES ==\n'
pkg list-installed 2>/dev/null | grep -E '^(termux-api|openssh|python|git|tailscale)' || true
printf '\n== SSH SERVICE ==\n'
pgrep -a sshd || true
printf '\n== STORAGE ==\n'
df -h "$HOME"
printf '\n== POWER ==\n'
termux-battery-status 2>/dev/null || true
```

Do not paste API keys, private keys, phone numbers, SMS contents, account tokens, or full environment dumps. The audit does not need them.
