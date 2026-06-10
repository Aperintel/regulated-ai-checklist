# Regulated AI Deployment Checklist

[![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg)](LICENSE)

A practitioner's pre-deployment checklist for shipping AI systems in regulated environments. Covers governance, model evaluation, audit, security, data protection, monitoring, incident response, supply chain, documentation, agentic AI, GPAI obligations, and ongoing change management. Drawn from operational practice across UK and EU regulated industries (FCA-supervised firms, NHS deployments, EU AI Act high-risk systems, financial services, public sector).

**EU AI Act enforcement note:** Obligations for high-risk AI systems under EU AI Act Annex III take full effect from **2 August 2026**. GPAI model provider obligations under Articles 53 and 55 applied from **2 August 2025**. Where a checklist item maps to a specific EU AI Act article, the article is noted in brackets. Items marked `[GDPR Art. X]` apply under both UK GDPR and EU GDPR.

This list is meant to be **used** during the design, build, and pre-deployment phases of any AI system that touches a regulated workflow. Print it, fork it, run through it before the buyer's compliance officer does, and bring the completed version to procurement.

Maintained by [Aperintel](https://github.com/aperintel). Community PRs welcome.

## How to use

- Copy the checklist into your own repo or wiki.
- Run through each section with the relevant function (engineering, compliance, security, data protection, operations).
- Tick items honestly. The point of the list is to surface the things you have not done; ticking a box you have not actually done defeats the purpose.
- Any item you cannot tick yet becomes a deployment gate. Buyers reading this list expect every item to be either ticked or have a written justification for being out of scope.
- Phases are roughly sequential by the lifecycle milestone in each header, but several phases (especially Phase 0 and Phase 6) must be revisited throughout the project, not just once at the stated point.

The checklist is licensed CC BY 4.0; you can fork it, customise it for your domain, and republish, with attribution.

---

## Phase 0: Governance foundations

Before any code is written.

- [ ] **Use case is named, scoped, and documented.** Specifically: what decision the AI is contributing to, what evidence the AI is using, what action follows the AI output, and which human (if any) authorises that action. `[EU AI Act Art. 9, Art. 13]`
- [ ] **Regulatory category is identified.** EU AI Act high-risk (Annex III)? Limited-risk (Art. 50)? FCA Consumer Duty in scope? NHS DSPT applicable? PRA/FCA model risk management expectations apply? Be specific about which articles or clauses apply to your deployment, not just "probably regulated".
- [ ] **Accountable individual is named.** A specific person at the deploying organisation is accountable for outcomes. Their name appears in the deployment record. The role is not "the AI team"; it is one person. `[EU AI Act Art. 17]`
- [ ] **Risk assessment is filed.** Document the foreseeable harms, the populations affected, the severity, and the mitigations. Update at every material change to the system. `[EU AI Act Art. 9]`
- [ ] **Decision to deploy is in writing.** Someone with the authority to commit the organisation to this AI system has signed off, in writing, on the risk assessment. The signoff sits on file. `[EU AI Act Art. 17]`
- [ ] **Fundamental Rights Impact Assessment is scoped.** For deployers of high-risk AI systems, a FRIA is required before deployment. Covers the impact on the right to human dignity, data protection, non-discrimination, consumer protection, and the rights of the child. Public-sector deployers and private bodies providing public services are specifically in scope. `[EU AI Act Art. 27]`
- [ ] **Technical documentation is drafted.** For providers of high-risk AI systems, technical documentation per Annex IV must be prepared before the system is placed on the market or put into service. Covers system description, training data, validation, intended purpose, risk management, and monitoring. `[EU AI Act Art. 11]`
- [ ] **Conformity assessment pathway is identified.** For providers of high-risk AI systems, the applicable conformity assessment procedure (internal control per Art. 43 or third-party assessment via notified body per Art. 44) is identified and initiated, and the EU declaration of conformity is planned. `[EU AI Act Art. 43, Art. 48]`
- [ ] **AI system inventory is updated.** The organisation maintains a register of AI systems in use. This deployment is added to that register with its classification, responsible owner, and regulatory category before the first line of production code is written. `[EU AI Act Art. 17]`

## Phase 1: Audit and provenance

Before the first production call.

- [ ] **Every AI call writes an audit entry.** Every model invocation, every prompt, every response, every guardrail decision is recorded. Default-on; not opt-in. `[EU AI Act Art. 12]`
- [ ] **Audit entries are tamper-evident.** The audit log is hash-chained, append-only, and cryptographically signed. A regulator asking "did you change this entry" can be answered with mathematical confidence, not assertion. `[EU AI Act Art. 12]`
- [ ] **Audit entries include enough to reconstruct the decision.** At minimum: input fingerprint, model identity and version, prompt template, response, timestamp, requester identity, policy decisions taken on the call. `[EU AI Act Art. 12]`
- [ ] **Model version and release identifier are captured in every audit entry.** A model update that happens mid-deployment must be detectable in the audit log. Post-incident reviews must be able to identify which model version produced which output. `[EU AI Act Art. 12]`
- [ ] **Audit log retention period is set.** Defined in writing, aligned to the relevant regulation (GDPR has its own clock; FCA SYSC 9.1 has its own; sector-specific rules may extend). Automated deletion runs on the same clock. `[EU AI Act Art. 19]`
- [ ] **Audit log is independently verifiable.** A third party (an auditor, a regulator, the buyer's internal audit team) can verify the chain without access to your systems.
- [ ] **Audit chain integrity is tested on a defined schedule.** The hash chain is verified programmatically, not assumed to be intact. Failures alert the operations team immediately. `[EU AI Act Art. 12]`
- [ ] **Provenance for training data is documented.** Where the model was trained, on what data, when, and what licence applies. "We use a third-party API" requires the API vendor's training-data provenance, too. `[EU AI Act Art. 10]`
- [ ] **AI system Software Bill of Materials (SBOM) is maintained.** All model weights, adapters, third-party libraries, and data pipelines that form part of the AI system are listed, with version pins. Updated on every dependency change.

## Phase 2: Model evaluation and validation

Before the model serves production traffic.

- [ ] **Accuracy is benchmarked on a representative production distribution.** Not just the vendor's published benchmark. A held-out evaluation set drawn from the same distribution as production inputs. The result is a number on file, not a verbal assurance. `[EU AI Act Art. 15]`
- [ ] **Accuracy is measured per demographic subgroup.** Overall accuracy figures mask subgroup-level failure. Measure performance separately for each subgroup that is material to the use case and document the results. `[EU AI Act Art. 9, Art. 10]`
- [ ] **Test set is free from training data contamination.** The evaluation set has been verified to exclude data used in training, fine-tuning, or validation during model development. Contaminated test sets produce inflated accuracy numbers.
- [ ] **Distribution shift between test set and production is assessed.** If the evaluation set was drawn from a different time period, geography, or population than production traffic will be, the gap is documented and its implications for expected accuracy are stated explicitly.
- [ ] **Failure modes are catalogued.** The types of errors the model makes are documented: false positives, false negatives, systematic biases, edge-case failures, and safety-critical failure modes. This catalogue is reviewed by the accountable individual. `[EU AI Act Art. 9]`
- [ ] **Model output calibration is assessed for generative systems.** Where the model produces confidence scores or probabilities, calibration is measured: do the stated probabilities correspond to empirical accuracy? Overconfident models that claim 95% confidence on wrong answers are a deployment risk. `[EU AI Act Art. 15]`
- [ ] **Hallucination rate is quantified for generative outputs.** For any generative model producing factual claims, the rate of confabulation on a domain-relevant evaluation set is measured and documented. Acceptable thresholds are set in writing before deployment, not after. `[EU AI Act Art. 15]`
- [ ] **Adversarial test suite covers model-specific attack classes.** Beyond prompt injection: model inversion (can the model reveal training data?), membership inference (can an attacker determine whether a specific record was in the training set?), and evasion attacks (can adversarially crafted inputs produce systematically wrong outputs?) are all tested. `[EU AI Act Art. 15]`
- [ ] **Red-team exercise is complete.** A structured human-led attack simulation has been run against the deployed system, not just the model in isolation. Findings are documented and mitigations are evidenced. `[EU AI Act Art. 15]`
- [ ] **Model card is complete before any pilot.** Training data summary, intended purpose, known limitations, known failure modes, out-of-scope uses, recommended human oversight levels. If a third-party model is used, the provider's model card has been reviewed and gaps have been noted. `[EU AI Act Art. 13]`

## Phase 3: Security and access control

Before the first user logs in.

- [ ] **Credentials are vaulted.** No API keys, signing keys, or service-account secrets in code, in repos, in CI variables, or in Slack. All credentials live in a managed vault. `[EU AI Act Art. 9]`
- [ ] **Credentials are rotated on a schedule.** Quarterly minimum for production. The rotation flow is automated; if it depends on a human remembering, it does not count.
- [ ] **Access is role-based.** Each role has the minimum permissions it needs. No standing admin access for engineers; access is requested, granted, and audit-logged per task.
- [ ] **Multi-factor authentication is enforced** on every account that touches the deployment, including third-party SaaS consoles.
- [ ] **Network segmentation is in place.** The AI workload runs in a network segment with the minimum egress it needs. Outbound calls to model providers go through a recorded proxy.
- [ ] **Prompt-injection guardrails are tested.** Adversarial inputs that try to bypass your guardrails are part of the test suite, not just hand-curated examples. `[EU AI Act Art. 15]`
- [ ] **Data exfiltration paths are reviewed.** Document every path that could exfiltrate sensitive data from the AI workflow (model provider logs, debugging tools, error reporting, audit log itself if shared externally).
- [ ] **Model supply chain is assessed.** If pre-trained weights or adapters are used, their provenance is documented: source, training data summary, licence, and the chain of custody from the original author to your deployment. `[EU AI Act Art. 10]`
- [ ] **Rate limiting and abuse detection are in place.** The AI endpoint cannot be driven at arbitrary volume by a single user or automated client. Anomalous call patterns alert the operations team.
- [ ] **AI and ML package vulnerabilities are scanned.** The CI pipeline includes dependency scanning that covers ML-specific packages (model serving frameworks, inference libraries, data processing libraries). Known CVEs in the dependency tree are tracked and patched on the same schedule as other software vulnerabilities.

## Phase 4: Data protection and compliance

Before the first piece of personal data touches the system.

- [ ] **Data processing impact assessment (DPIA) is complete.** Required under UK and EU GDPR for high-risk processing. AI making decisions about individuals almost always qualifies. `[GDPR Art. 35]`
- [ ] **Lawful basis for processing is identified.** Per UK and EU GDPR Article 6. Document the basis per data category, not just per system. `[GDPR Art. 6]`
- [ ] **Data minimisation is applied.** The AI sees only the data it needs. Personal data unrelated to the decision is filtered out before it reaches the model. `[GDPR Art. 5(1)(c), EU AI Act Art. 10]`
- [ ] **PII redaction runs on inbound data.** PII that the AI does not need is redacted at the boundary. The redaction is recorded in the audit log.
- [ ] **Special category data handling is explicitly assessed.** Processing of health, biometric, genetic, racial or ethnic origin, or other special category data (Art. 9 GDPR) requires an explicit lawful basis beyond the standard Art. 6 bases. Document the specific basis and the safeguards applied. `[GDPR Art. 9]`
- [ ] **Children's data protections are in scope if minors may be affected.** Where the system may process data relating to or make decisions affecting individuals under 18, the additional protections under UK GDPR Art. 8, the Age Appropriate Design Code (UK), and COPPA (US) are assessed and documented.
- [ ] **Equality and non-discrimination obligations are mapped to the model's decision scope.** Under the UK Equality Act 2010, decisions that have a discriminatory effect on individuals with protected characteristics (age, disability, gender reassignment, marriage and civil partnership, pregnancy and maternity, race, religion or belief, sex, sexual orientation) carry legal exposure. For public bodies, the Public Sector Equality Duty (Section 149) applies. Document which protected characteristics are material to the use case.
- [ ] **Right to erasure is implementable.** When a data subject requests erasure, you can identify, locate, and erase their data from every system component, including the audit log (within the constraints of the legal-obligation lawful-basis exception). `[GDPR Art. 17]`
- [ ] **Right to explanation is implementable.** A data subject affected by an automated decision can receive a meaningful explanation of the logic involved. `[GDPR Art. 22, EU AI Act Art. 13]`
- [ ] **International data transfers are documented.** Every transfer outside the operator's jurisdiction is documented with the legal mechanism (adequacy, standard contractual clauses, etc.). If your model provider hosts in a different jurisdiction, this applies to every call. `[GDPR Art. 44-49]`
- [ ] **Consent records are durable.** If consent is the lawful basis for any processing path, the consent record is signed, timestamped, and as tamper-evident as the audit log. `[GDPR Art. 7]`

## Phase 5: Operational monitoring

Before the system goes to first paying customer.

- [ ] **Drift detection is live.** Model outputs are monitored for distributional shift. Alerts fire on anomalies. `[EU AI Act Art. 72]`
- [ ] **Concept drift is distinguished from data drift in monitoring.** Data drift (the input distribution has changed) and concept drift (the relationship between inputs and correct outputs has changed) require different responses. Your monitoring system detects and categorises both.
- [ ] **Bias monitoring is live.** Output distributions are tracked across demographic subgroups relevant to the use case. Statistically significant divergence triggers a review, not just a log entry. `[EU AI Act Art. 9, Art. 10]`
- [ ] **Cost monitoring is live.** Per-call cost, per-feature cost, and per-customer cost are visible to operators in near-real-time. A runaway loop does not become a six-figure bill.
- [ ] **Latency monitoring is live.** P95 latency is tracked and alerted. SLAs to your buyers should match what you actually deliver under load.
- [ ] **Error reporting is wired to the audit log.** Every error condition that affects a decision is recorded with enough context for a post-incident review.
- [ ] **Output quality is sampled.** A statistically meaningful sample of outputs is reviewed by a human on a defined cadence. The sample size is justified statistically, not chosen arbitrarily. The review results feed back into model selection and prompt engineering. `[EU AI Act Art. 72]`
- [ ] **Feedback loops are detected and monitored.** Where model outputs influence future inputs (recommendations that shape user behaviour, decisions that affect the data pool the model is later trained on), the loop is documented and monitored for amplification effects.
- [ ] **Override patterns are tracked.** When a human overrides an AI output, the override is recorded, categorised, and reviewed. A rising override rate is a leading indicator that the model has drifted. `[EU AI Act Art. 14]`

## Phase 6: Incident response

Before you announce general availability.

- [ ] **Incident playbook is written.** What happens when a regulator phones at 09:00 on a Monday asking about a specific decision. Who is paged. What artefacts are produced. What the customer is told. Practised in writing; ideally in a drill. `[EU AI Act Art. 17, Art. 73]`
- [ ] **Decision-rollback path exists.** When you discover that a model was producing wrong outputs for a specific cohort, you can identify the affected decisions, freeze further AI processing for that cohort, and surface a list to the operator within hours, not weeks.
- [ ] **Kill switch is operational.** A single control disables AI decision-making for the affected scope. The kill switch is tested quarterly. The action is audit-logged. `[EU AI Act Art. 14]`
- [ ] **Breach notification clock is understood.** Under UK and EU GDPR you have 72 hours to notify the relevant regulator. The on-call team knows this, and knows what counts as a "breach" in your specific system. `[GDPR Art. 33]`
- [ ] **Serious incident notification is understood for EU AI Act.** For high-risk AI systems, serious incidents affecting health, safety, or fundamental rights must be reported to national market surveillance authorities. Standard notification deadline is 15 days; incidents involving suspected deaths have a 10-day deadline. `[EU AI Act Art. 73]`
- [ ] **Regulator liaison process is defined.** The person who speaks to the ICO, FCA, PRA, or national market surveillance authority in an incident is named. What they are authorised to disclose without further sign-off is written down. What must go to legal first is written down.
- [ ] **Post-incident review process is documented.** After every serious incident or near-miss, a structured review is conducted, the root cause is documented, and the findings are fed into the risk assessment and the system design. `[EU AI Act Art. 17]`
- [ ] **Near-miss reporting channel exists.** Operators and users have a way to report close calls (outputs that were nearly harmful, guardrails that nearly failed, edge cases that revealed a gap) without it requiring a formal incident. Near-misses are reviewed on the same cadence as incidents.
- [ ] **Communications template is drafted.** Pre-drafted external comms for the most likely incident types. Reviewed by legal. Avoids the cold-start problem at the worst possible moment.

## Phase 7: Vendor and supply chain

Before you sign with each external provider.

- [ ] **Every external provider is named on a register.** Model providers, hosting providers, observability providers, vault providers, audit-log providers, every dependency. `[EU AI Act Art. 25]`
- [ ] **Each provider has a DPA on file.** A signed Data Processing Agreement under UK and EU GDPR. Filed where you can find it during an audit. `[GDPR Art. 28]`
- [ ] **Each provider has known regulatory posture.** ISO 27001? SOC 2 Type II? ISO 42001? Document the certifications and the audit reports you rely on.
- [ ] **Model card or equivalent disclosure exists for every model used.** Training data summary, known limitations, intended use, known failure modes. If the provider does not publish one, document what they have disclosed and note the gap. `[EU AI Act Art. 13]`
- [ ] **Open-source licence compliance is audited.** Every open-source model, dataset, and library in your AI stack is reviewed for licence compatibility with commercial use. GPL, AGPL, and model-specific research-only licences require explicit clearance before production deployment.
- [ ] **AI-specific SLAs are in every model provider contract.** Uptime SLAs are not enough. Contracts should address: minimum accuracy floors, notification requirements when model versions change, data retention and deletion guarantees, and the provider's liability for model-induced errors affecting your customers.
- [ ] **Right-to-audit clause is in every AI supplier contract.** Your organisation can request evidence of the supplier's AI Act compliance, training data documentation, and security posture. The clause covers both your own audits and those conducted by regulators or third-party auditors on your behalf.
- [ ] **Sub-processor list is reviewed.** Each provider's sub-processors are reviewed. Material additions trigger a re-review. `[GDPR Art. 28]`
- [ ] **Provider continuity is planned for.** If the model provider is acquired, deprecates the model, or goes down, you have a documented plan that does not start with "panic".

## Phase 8: Documentation and training

Before the team that operates the system starts shifts.

- [ ] **System card is published internally.** What the system does, what it does not do, known limitations, known failure modes, sensitive-context guidance. `[EU AI Act Art. 13]`
- [ ] **Operator runbook is published.** Daily operations, common failure modes, escalation paths, regulator-conversation playbook.
- [ ] **Training is delivered.** Every operator who interacts with the AI workflow has been trained on the system card and the runbook. Refresh cadence is defined. `[EU AI Act Art. 26]`
- [ ] **End-user-facing documentation exists.** Where appropriate, the people affected by the AI's decisions can find documentation that meets the AI Act Article 13 transparency obligations. `[EU AI Act Art. 13]`
- [ ] **Plain-language summary is published for non-technical stakeholders.** A version of the system card and the key risk findings that a non-engineer (a board member, a regulator, a customer) can read and act on. Separate from the technical documentation.
- [ ] **Chatbot and synthetic-voice systems declare their AI nature.** Where users interact with an AI through text, voice, or video (chatbots, voice assistants, deepfakes), the AI nature of the interaction is disclosed. Not in the small print; at the point of contact. `[EU AI Act Art. 50]`
- [ ] **Change management process is defined.** What counts as a material change to the AI system, who has authority to approve it, and what re-assessments it triggers are written down before go-live, not resolved ad hoc when a change request arrives. `[EU AI Act Art. 17]`
- [ ] **Compliance team has a stakeholder seat.** The compliance team is consulted on material changes to the system, the model, or the deployment scope. Not asked for sign-off after the fact.

## Phase 9: Agentic AI systems

Apply this phase when the AI system can plan across multiple steps, call external tools, execute actions in external systems, or operate across multiple agents with limited per-step human involvement.

- [ ] **Every agent action is logged with its principal.** The initiating user, the tool called, the inputs passed, and the outcome are recorded per action. An agent that calls ten tools in one session produces ten audit entries. `[EU AI Act Art. 12]`
- [ ] **Tool permissions follow least privilege.** Each agent is granted only the tool access it needs for its defined scope. Permissions are reviewed before deployment and after any change to the agent's role.
- [ ] **Agent scope is bounded in writing.** A document states what actions each agent is permitted to take, what systems it can write to, and what it cannot do. The boundary is enforced at the tool layer, not just by instruction.
- [ ] **Human-in-the-loop gates exist for irreversible actions.** Any agent action that cannot be undone (sending an email, making a payment, modifying a database record) requires explicit human approval before execution. The approval is logged as a first-class audit event. `[EU AI Act Art. 14]`
- [ ] **Multi-agent handoffs are audited.** When one agent passes control or context to another, the handoff is logged: source agent, destination agent, context fingerprint, and timestamp. The full chain of custody is reconstructable from the audit log.
- [ ] **Agent-to-agent communication is authenticated.** An agent receiving instructions from another agent verifies the source before acting. Unauthenticated instructions are rejected and logged.
- [ ] **Agent behaviour under adversarial conditions is tested.** What happens when a tool returns malicious content designed to manipulate the agent's next action? What happens when an orchestrating agent sends out-of-scope instructions? Both scenarios are in the test suite.
- [ ] **Resource consumption limits are enforced per agent.** Token budget, external API call budget, and wall-clock time budget are capped per agent session. A runaway agent loop does not consume unbounded resources.
- [ ] **A kill switch exists at agent, workflow, and system level.** A single control can stop one agent, one workflow, or all agentic activity. Each level is tested and the test result is logged. `[EU AI Act Art. 14]`
- [ ] **Memory and context retention is privacy-assessed.** If the agent retains information across sessions (user preferences, prior decisions, learned facts), the retention period, storage location, and erasure mechanism are documented and assessed under the DPIA. `[GDPR Art. 35]`
- [ ] **Tool output is validated before the agent acts on it.** The agent does not pass tool output directly to the next step without a sanity check. Unexpected or malformed tool output is flagged rather than silently processed.
- [ ] **Prompt injection is tested across all tool inputs.** Adversarial content injected through any tool return value (search results, database reads, API responses, email content) is part of the red-team suite. `[EU AI Act Art. 15]`

## Phase 10: General-purpose AI (GPAI) model obligations

Apply this phase when your system is built on a foundation model or large language model accessed via API, or when your organisation is a provider of a GPAI model. GPAI obligations under the EU AI Act applied from **2 August 2025**.

- [ ] **GPAI model classification is established.** Determine whether the underlying model qualifies as a general-purpose AI model under EU AI Act Article 51. The systemic risk threshold is 10^25 FLOPs of training compute. If the model meets this threshold, additional obligations under Art. 55 apply. `[EU AI Act Art. 51]`
- [ ] **Provider's technical documentation and usage policy are reviewed and retained.** GPAI model providers are required to make technical documentation, training data summaries, and usage policies available to downstream deployers. Obtain and file this documentation before building on the model. `[EU AI Act Art. 53]`
- [ ] **Copyright and training data summary from the GPAI provider is on file.** Providers of GPAI models must publish a summary of training data used, including which copyright exceptions (e.g. text and data mining) were relied on. Retain this summary as part of your vendor documentation. `[EU AI Act Art. 53]`
- [ ] **Deployer obligations on top of the GPAI provider's are documented.** The GPAI provider's EU AI Act compliance covers the model layer; it does not cover your deployment layer. Document explicitly what the provider's compliance does and does not cover, and what additional obligations fall on your organisation as deployer. `[EU AI Act Art. 25, Art. 26]`
- [ ] **Model version changes from the GPAI provider are monitored.** Foundation model providers can push capability or alignment changes without advance notice. A process exists to detect model updates, assess their impact on your deployment's compliance posture, and trigger a re-assessment where necessary.
- [ ] **For systemic-risk GPAI providers: adversarial testing results are reviewed.** Providers of GPAI models with systemic risk must conduct adversarial testing (red-teaming) and report serious incidents to the EU AI Office. If your deployment depends on such a model, review the provider's published adversarial testing programme and incident history. `[EU AI Act Art. 55]`
- [ ] **Terms of service and API contract are reviewed for AI Act compliance obligations.** The provider's ToS must be compatible with your own EU AI Act obligations as a deployer. Key areas: training on your prompts and outputs, data retention and deletion, subprocessor disclosures, liability allocation for model-induced errors.

## Phase 11: Change management and ongoing compliance

Apply on every material change to the AI system, and on a scheduled annual review regardless of whether a change has occurred.

- [ ] **Change register is maintained.** Every material change to the AI system (model update, prompt change, guardrail modification, data pipeline change, scope expansion, new user cohort) is logged with date, description, and the name of the person who approved it. `[EU AI Act Art. 17]`
- [ ] **Material change criteria are defined in writing.** What counts as a "significant modification" requiring a full re-run of relevant checklist phases is documented before the first change request arrives. EU AI Act Art. 16(d) requires updated technical documentation on significant modifications; the criteria for what constitutes "significant" should be set internally, not argued about after the fact. `[EU AI Act Art. 18]`
- [ ] **Model updates trigger a re-run of Phase 2.** Any change to the underlying model (version bump, fine-tune, quantisation change, adapter swap) triggers a fresh run of the model evaluation and validation phase. The re-evaluation results are filed in the change register.
- [ ] **Scope changes trigger a re-run of Phase 0 and Phase 4.** Extending the system to new user cohorts, new decision types, or new jurisdictions is treated as a new deployment for governance and data protection purposes.
- [ ] **Post-market monitoring feeds into the change assessment.** Drift alerts, bias divergence, rising override rates, and near-miss reports are all evaluated as potential triggers for a re-assessment, not just as operational metrics. `[EU AI Act Art. 72]`
- [ ] **Regulatory change is monitored and assigned.** A named person reads and summarises relevant regulatory publications (ICO guidance updates, FCA publications, EU AI Office guidance, PRA/FCA model risk management updates) as they are issued. New obligations identified are mapped to this checklist and added as items.
- [ ] **Annual review is scheduled regardless of triggering events.** Even with no material changes, the full checklist is reviewed annually. The review is dated, the reviewer is named, and the outcome (no action required, or list of actions) is filed.
- [ ] **Deprecation and decommissioning plan is documented.** When and under what conditions the AI system will be shut down or replaced. What happens to the audit logs, the data, and the model artefacts after shutdown. Retention obligations survive decommissioning. `[EU AI Act Art. 19]`

---

## Contributing

PRs welcome. Add items only when an item materially improves a regulated-AI deployment's defensibility. Proposed items should name the regulatory obligation or operational risk they address. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

CC BY 4.0. You can fork, customise, and republish this list with attribution to [Aperintel](https://github.com/aperintel) and a link back to the canonical version.
