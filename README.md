# Regulated AI Deployment Checklist

[![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg)](LICENSE)

A practitioner's pre-deployment checklist for shipping AI systems in regulated environments. Covers governance, audit, security, data protection, monitoring, incident response, and supply chain. Drawn from operational practice across UK and EU regulated industries (FCA-supervised firms, NHS deployments, EU AI Act high-risk systems).

This list is meant to be **used** during the design and pre-deployment phases of an AI system that touches a regulated workflow. Print it, fork it, run through it before the buyer's compliance officer does, and bring the completed version to procurement.

Maintained by [Aperintel](https://github.com/aperintel). Community PRs welcome.

## How to use

- Copy the checklist into your own repo or wiki.
- Run through each section with the relevant function (engineering, compliance, security, data protection, operations).
- Tick items honestly. The point of the list is to surface the things you have not done; ticking a box you have not actually done defeats the purpose.
- Any item you cannot tick yet becomes a deployment gate. Buyers reading this list expect every item to be either ticked or have a written justification for being out of scope.

The checklist is licensed CC BY 4.0; you can fork it, customise it for your domain, and republish, with attribution.

---

## Phase 0: Governance foundations

Before any code is written.

- [ ] **Use case is named, scoped, and documented.** Specifically: what decision the AI is contributing to, what evidence the AI is using, what action follows the AI output, and which human (if any) authorises that action.
- [ ] **Regulatory category is identified.** EU AI Act high-risk? Limited-risk? FCA Consumer Duty in scope? NHS DSPT applicable? Be specific about which articles or clauses apply.
- [ ] **Accountable individual is named.** A specific person at the deploying organisation is accountable for outcomes. Their name appears in the deployment record. The role is not "the AI team"; it is one person.
- [ ] **Risk assessment is filed.** Document the foreseeable harms, the populations affected, the severity, and the mitigations. Update at every material change to the system.
- [ ] **Decision to deploy is in writing.** Someone with the authority to commit the organisation to this AI system has signed off, in writing, on the risk assessment. The signoff sits on file.

## Phase 1: Audit and provenance

Before the first production call.

- [ ] **Every AI call writes an audit entry.** Every model invocation, every prompt, every response, every guardrail decision is recorded. Default-on; not opt-in.
- [ ] **Audit entries are tamper-evident.** The audit log is hash-chained, append-only, and cryptographically signed. A regulator asking "did you change this entry" can be answered with mathematical confidence, not assertion.
- [ ] **Audit entries include enough to reconstruct the decision.** At minimum: input fingerprint, model identity, prompt template, response, timestamp, requester identity, policy decisions taken on the call.
- [ ] **Audit log retention period is set.** Defined in writing, aligned to the relevant regulation (GDPR has its own clock; FCA SYSC 9.1 has its own; sector-specific rules may extend). Automated deletion runs on the same clock.
- [ ] **Audit log is independently verifiable.** A third party (an auditor, a regulator, the buyer's internal audit team) can verify the chain without access to your systems.
- [ ] **Provenance for training data is documented.** Where the model was trained, on what data, when, and what licence applies. "We use a third-party API" requires the API vendor's training-data provenance, too.

## Phase 2: Security and access control

Before the first user logs in.

- [ ] **Credentials are vaulted.** No API keys, signing keys, or service-account secrets in code, in repos, in CI variables, or in Slack. All credentials live in a managed vault.
- [ ] **Credentials are rotated on a schedule.** Quarterly minimum for production. The rotation flow is automated; if it depends on a human remembering, it does not count.
- [ ] **Access is role-based.** Each role has the minimum permissions it needs. No standing admin access for engineers; access is requested, granted, and audit-logged per task.
- [ ] **Multi-factor authentication is enforced** on every account that touches the deployment, including third-party SaaS consoles.
- [ ] **Network segmentation is in place.** The AI workload runs in a network segment with the minimum egress it needs. Outbound calls to model providers go through a recorded proxy.
- [ ] **Prompt-injection guardrails are tested.** Adversarial inputs that try to bypass your guardrails are part of the test suite, not just hand-curated examples.
- [ ] **Data exfiltration paths are reviewed.** Document every path that could exfiltrate sensitive data from the AI workflow (model provider logs, debugging tools, error reporting, audit log itself if shared externally).

## Phase 3: Data protection and compliance

Before the first piece of personal data touches the system.

- [ ] **Data processing impact assessment (DPIA) is complete.** Required under UK and EU GDPR for high-risk processing. AI making decisions about individuals almost always qualifies.
- [ ] **Lawful basis for processing is identified.** Per UK and EU GDPR Article 6. Document the basis per data category, not just per system.
- [ ] **Data minimisation is applied.** The AI sees only the data it needs. Personal data unrelated to the decision is filtered out before it reaches the model.
- [ ] **PII redaction runs on inbound data.** PII that the AI does not need is redacted at the boundary. The redaction is recorded in the audit log.
- [ ] **Right to erasure is implementable.** When a data subject requests erasure, you can identify, locate, and erase their data from every system component, including the audit log (within the constraints of the legal-obligation lawful-basis exception).
- [ ] **Right to explanation is implementable.** A data subject affected by an automated decision can receive a meaningful explanation of the logic involved. Under UK and EU GDPR Article 22 plus AI Act Article 13 for high-risk systems.
- [ ] **International data transfers are documented.** Every transfer outside the operator's jurisdiction is documented with the legal mechanism (adequacy, standard contractual clauses, etc.). If your model provider hosts in a different jurisdiction, this applies to every call.
- [ ] **Consent records are durable.** If consent is the lawful basis for any processing path, the consent record is signed, timestamped, and as tamper-evident as the audit log.

## Phase 4: Operational monitoring

Before the system goes to first paying customer.

- [ ] **Drift detection is live.** Model outputs are monitored for distributional shift. Alerts fire on anomalies.
- [ ] **Cost monitoring is live.** Per-call cost, per-feature cost, and per-customer cost are visible to operators in near-real-time. A runaway loop does not become a six-figure bill.
- [ ] **Latency monitoring is live.** P95 latency is tracked and alerted. SLAs to your buyers should match what you actually deliver under load.
- [ ] **Error reporting is wired to the audit log.** Every error condition that affects a decision is recorded with enough context for a post-incident review.
- [ ] **Output quality is sampled.** A statistically meaningful sample of outputs is reviewed by a human on a defined cadence. The review results feed back into model selection and prompt engineering.
- [ ] **Override patterns are tracked.** When a human overrides an AI output, the override is recorded, categorised, and reviewed. A rising override rate is a leading indicator that the model has drifted.

## Phase 5: Incident response

Before you announce general availability.

- [ ] **Incident playbook is written.** What happens when a regulator phones at 09:00 on a Monday asking about a specific decision. Who is paged. What artefacts are produced. What the customer is told. Practised in writing; ideally in a drill.
- [ ] **Decision-rollback path exists.** When you discover that a model was producing wrong outputs for a specific cohort, you can identify the affected decisions, freeze further AI processing for that cohort, and surface a list to the operator within hours, not weeks.
- [ ] **Kill switch is operational.** A single control disables AI decision-making for the affected scope. The kill switch is tested quarterly. The action is audit-logged.
- [ ] **Breach notification clock is understood.** Under UK and EU GDPR you have 72 hours to notify the relevant regulator. The on-call team knows this, and knows what counts as a "breach" in your specific system.
- [ ] **Communications template is drafted.** Pre-drafted external comms for the most likely incident types. Reviewed by legal. Avoids the cold-start problem at the worst possible moment.

## Phase 6: Vendor and supply chain

Before you sign with each external provider.

- [ ] **Every external provider is named on a register.** Model providers, hosting providers, observability providers, vault providers, audit-log providers, every dependency.
- [ ] **Each provider has a DPA on file.** A signed Data Processing Agreement under UK and EU GDPR. Filed where you can find it during an audit.
- [ ] **Each provider has known regulatory posture.** ISO 27001? SOC 2 Type II? ISO 42001? Document the certifications and the audit reports you rely on.
- [ ] **Sub-processor list is reviewed.** Each provider's sub-processors are reviewed. Material additions trigger a re-review.
- [ ] **Provider continuity is planned for.** If the model provider is acquired, deprecates the model, or goes down, you have a documented plan that does not start with "panic".

## Phase 7: Documentation and training

Before the team that operates the system starts shifts.

- [ ] **System card is published internally.** What the system does, what it does not do, known limitations, known failure modes, sensitive-context guidance.
- [ ] **Operator runbook is published.** Daily operations, common failure modes, escalation paths, regulator-conversation playbook.
- [ ] **Training is delivered.** Every operator who interacts with the AI workflow has been trained on the system card and the runbook. Refresh cadence is defined.
- [ ] **End-user-facing documentation exists.** Where appropriate, the people affected by the AI's decisions can find documentation that meets the AI Act Article 13 transparency obligations.
- [ ] **Compliance team has a stakeholder seat.** The compliance team is consulted on material changes to the system, the model, or the deployment scope. Not asked for sign-off after the fact.

---

## Contributing

PRs welcome. Add items only when an item materially improves a regulated-AI deployment's defensibility. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

CC BY 4.0. You can fork, customise, and republish this list with attribution to [Aperintel](https://github.com/aperintel) and a link back to the canonical version.
