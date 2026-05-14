# R2 — Israeli compliance roadmap (PPL Amendment 13 + AMAR SaMD)

Complete regulatory map for a clinical Hebrew SLP product (SaMD) deployed in Israel. STT layer is Soniox via EU region with DPA + patient consent. Patient audio + transcripts + clinical features extracted from them are processed.

Two regimes apply:
- **(a) Israeli PPL Amendment 13** (effective Aug 14, 2025) for data privacy.
- **(b) Israeli Ministry of Health AMAR/AMIR pathway** for software-as-medical-device.

---

## 1. PPL Amendment 13 — core obligations

Effective **14 Aug 2025**. "Highly Sensitive Information" (matching GDPR Art. 9) explicitly includes **health data, genetic data, biometric data, and information about mental/physical condition** — patient audio + clinical features qualify.

- **DPO trigger:** mandatory when entity is a public body, processes "highly sensitive" data on ≥500,000 data subjects, or systematically monitors data subjects on a large scale. As a database owner processing health data, threshold can be met well below 500k depending on processing nature.
- **DPIA trigger:** required for processing likely to result in high risk to data subjects — large-scale sensitive-data processing, automated decision-making, and new technologies all qualify. **Clinical AI = automatic trigger.**
- **Database Definition Document** (מסמך הגדרות מסד הנתונים): statutory record of processing — must contain: database purposes, data categories, data sources, recipients, transfer destinations, retention periods, security level (low/medium/high — health = high), and risk assessment summary. Must be approved by management and updated continuously.

Sources: [IAPP — Amendment 13 sweeping reform](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform); [Goldfarb Gross Seligman](https://www.goldfarb.com/amendment-13-to-the-israeli-privacy-protection-law/); [Gornitzky — 10 steps](https://www.gornitzky.com/privacy-protection-in-2025-10-steps-to-navigate-the-implementation-of-obligations/)

## 2. DPO requirements

- DPO **must be a natural person** (not a firm).
- Internal employee **or external contractor permitted**; small companies can outsource.
- Required expertise: Israeli privacy law fluency, infosec literacy, understanding of organisation's data flows.
- **Reports directly to CEO or senior executive**, must be independent.
- Disqualifying roles: CEO, CFO, CTO, IT Manager, Head of Marketing/Customer Success.
- Organisation must provide resources + access.

Sources: [Arnon Tadmor-Levy — PPA draft DPO clarification](https://arnontl.com/news/draft-clarification-by-israeli-privacy-protection-authority-on-new-requirements-for-appointment-of-data-protection-officers/); [INPLP](https://inplp.com/latest-news/article/new-amendment-to-israeli-privacy-protection-law-and-mandatory-dpo-appointment/)

## 3. Cross-border transfer

Israel has **no codified whitelist**; transfers permitted if destination ensures "adequate level" — in practice this means **EU/EEA and EU-adequacy-decision countries (UK included), plus countries with similar protection**.

- **EU = adequate.**
- **UK = adequate.**
- **US = NOT adequate** (no Privacy Shield equivalent; SCCs alone are not automatic — a DPA with PPL-aligned clauses is needed).
- Israel itself holds EU adequacy.
- **Soniox EU-region with DPA is the cleanest path.**
- Onward transfer from the importer requires equivalent restrictions.

Sources: [DLA Piper — Israel transfers](https://www.dlapiperdataprotection.com/index.html?t=transfer&c=IL); [Linklaters — Data Protected Israel](https://www.linklaters.com/en-us/insights/data-protected/data-protected---israel)

## 4. Patient consent

Must be **free, specific, informed, unambiguous**. Required disclosures under Amendment 13:
- Controller identity + contact
- Processing purposes
- Data categories
- Recipients/processors
- **Foreign transfer destinations**
- Retention period
- Data-subject rights (access/rectification/deletion)
- Consequences of refusal
- Withdrawal mechanism

**No statutory rule forcing Hebrew**, but PPA mandates inspection responses be provided in Hebrew/English/Arabic — practical norm for clinical consent is **Hebrew (plus Arabic where relevant)**. No official PPA template published; sector practice uses MoH informed-consent forms as scaffolding.

Sources: [Pandectes guide](https://pandectes.io/blog/understanding-israels-privacy-protection-law-and-its-requirements/); [TechPolicy.org.il overview](https://techpolicy.org.il/wp-content/uploads/2024/10/Overview-of-Amendment-no-13-FINAL-FINAL-FOR-UPLOAD-FOR-WEBSITE-COLLATED-1.pdf)

## 5. Encryption, logs, pen-test

Under **Privacy Protection (Data Security) Regulations 2017** — health databases = **high security level**.

- Encryption at rest + in transit
- Role-based access
- **Access logs retained ≥24 months** (Reg. 10)
- Documented database structure
- Incident-response playbook
- **Penetration tests + risk surveys every 18 months** for medium/high databases
- Logs must capture: identity of accessor, timestamp, action, affected records

Sources: [IAPP — Israeli data security regulations tutorial](https://iapp.org/news/a/the-new-israeli-data-security-regulations-a-tutorial); [BigID — Amendment 13](https://bigid.com/blog/what-israel-amendment-13-means-for-businesses-in-2025/)

## 6. Breach notification

- **Severe Security Incident** = unauthorised use or integrity harm of high-security DB (any data) or material part of medium-security DB.
- Notification to **PPA "immediately" — PPA guidance: 24h target, 72h hard ceiling**.
- PPA may then **order notification to affected data subjects**.
- Amendment 13 tightens this; financial-sector practice already runs the 72h clock.

Sources: [Baker McKenzie — Israel breach](https://resourcehub.bakermckenzie.com/en/resources/global-data-and-cyber-handbook/emea/israel/topics/security-requirements-and-breach-notification); [Pearl Cohen](https://www.pearlcohen.com/israel-privacy-protection-authority-hardens-its-policy-on-breach-notification-timescales/)

## 7. AMAR / AMIR registration

Hebrew SLP transcription + clinical-feature extraction *intended for diagnosis/monitoring/treatment of speech disorders* **= SaMD under the Medical Equipment Law 5772-2012**.

- **Israel does not run its own classification** — adopts reference-country class (FDA, EU MDR, Health Canada, TGA, UK MHRA).
- Likely **Class II / IIa** (clinical support, non-life-sustaining).
- **AMAR** = the MoH Medical Devices Division (the regulator).
- **AMIR** = the AMAR online registration portal ("AMAR-SAFE").
- Class I/A SaMD with prior reference-country clearance: **self-declaration, certificate within 48h**.
- Higher classes: full dossier review.
- Foreign manufacturers must appoint an **Israel Registration Holder (IRH)** with local ISO 9001.

Sources: [BIOREG — AMAR](https://bioregservices.com/regulatory-consulting-israel-amar/); [Emergo — AMAR registration](https://www.emergobyul.com/services/medical-device-registration-and-approval-israel); [Eran Yona — SaMD guideline](https://eranyona.com/samd-ivd-and-e-health-services-registration-in-israel-guideline/)

## 8. ISO 13485 + IEC 62304

- **ISO 13485:2016** required as QMS evidence for Class IIa+ devices (Class I = statement of compliance acceptable, no cert mandatory).
- **IEC 62304** (software lifecycle) is the de facto international standard AMAR accepts via reference-country approval — SLP analysis software is **Software Safety Class B** (non-serious injury possible).
- **No notified body in Israel** — AMAR relies on reference-country certification (CE notified body or FDA 510(k)).
- Path: get a CE certificate from an EU notified body (e.g., BSI, TÜV SÜD), then leverage that for AMAR.

Sources: [Compliance & Risks — Israel MDR guide](https://www.complianceandrisks.com/blog/a-comprehensive-guide-to-medical-device-regulation-in-israel-registration-documentation-and-labeling-requirements/); [Hardian Health — AI device standards](https://www.hardianhealth.com/insights/regulatory-ai-medical-device-standards)

## 9. Insurance / liability

- No statutory minimum specific to medical-device makers, but **IRH must demonstrate financial coverage** for defective-product liability and user injury in Israel.
- Standard market practice for clinical SaaS: **product liability + professional indemnity USD 1–5M per claim** scaling with device class and patient volume.
- Israeli Product Liability Law 5740-1980 imposes strict liability on manufacturers.

Sources: [Pearl Cohen — Pharma/MD regulation Israel](https://www.pearlcohen.com/wp-content/uploads/2022/10/2023-Pharma-Medical-Device-Regulation-Israel.pdf); [ICLG Insurance Israel](https://iclg.com/practice-areas/insurance-and-reinsurance-laws-and-regulations/israel)

## 10. Hospital procurement (Sheba/Hadassah/Loewenstein)

No public unified Israeli hospital procurement spec found. Practice norm (drawn from healthcare-vendor frameworks and the MoH cybersecurity directive for HMOs/hospitals) requires:

- **ISO 27001 cert**
- **SOC 2 Type II** (or HITRUST)
- Recent **pen-test report (<12 months)**
- Completed security questionnaire
- DPIA copy
- AMAR certificate
- Evidence of Hebrew-language UI + clinical validation
- IRB/Helsinki Committee approval for any data collection
- BAA-equivalent DPA
- Indemnity insurance certificate

Sheba ARC and Hadassah Innovation specifically prefer vendors with prior pilot exposure.

Sources: [Censinet — Healthcare vendor checklist](https://censinet.com/perspectives/the-ultimate-healthcare-vendor-risk-assessment-checklist-for-2025); [ICLG Digital Health Israel 2026](https://iclg.com/practice-areas/digital-health-laws-and-regulations/israel)

## 11. Sandbox

**Israel Innovation Authority "Regulatory Sandbox for Breakthrough AI Technologies in Healthcare"** — joint IIA + MoH. Combines grants + regulatory relief inside a controlled clinical pilot. Submission window currently open per the IIA call page.

Sources: [IIA — AI in Health Sandbox call](https://innovationisrael.org.il/en/calls_for_proposal/ai-in-health-sandbox/); [Cambridge Forum — Israeli AI sandbox](https://www.cambridge.org/core/journals/cambridge-forum-on-ai-law-and-governance/article/nascent-regulatory-sandbox-frameworks-for-ai-in-israel/C9E5B9630A678D809BDA109AE018881A)

---

## Critical gaps requiring lawyer

- **Exact DPO threshold** for our headcount/data volume — Amendment 13 text + PPA draft clarification both leave room.
- **SaMD intended-use statement** — wording determines class (decision support vs. diagnostic) and registration route.
- **Reference-country strategy** — FDA 510(k) vs CE MDR.
- **Israeli Product Liability + IRH contract** structure: who is on the hook in court.
- **Helsinki Committee (MoH form 11) approval** for any prospective clinical data — separate from AMAR.
- **Soniox DPA review** vs PPL Amendment 13 onward-transfer clauses (US parent entity risk).
- **Hebrew clinical-validation evidence threshold** — undefined in regulation, set per-hospital.
- **Sandbox eligibility opinion** — whether participating defers AMAR or runs in parallel.

## Sources

[IAPP — Amendment 13 sweeping reform](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform) · [IAPP — Israeli data security regulations tutorial](https://iapp.org/news/a/the-new-israeli-data-security-regulations-a-tutorial) · [Goldfarb Gross Seligman — Amendment 13](https://www.goldfarb.com/amendment-13-to-the-israeli-privacy-protection-law/) · [Gornitzky — Privacy 2025 10 steps](https://www.gornitzky.com/privacy-protection-in-2025-10-steps-to-navigate-the-implementation-of-obligations/) · [Arnon Tadmor-Levy — PPA DPO clarification](https://arnontl.com/news/draft-clarification-by-israeli-privacy-protection-authority-on-new-requirements-for-appointment-of-data-protection-officers/) · [Baker McKenzie — Israel breach notification](https://resourcehub.bakermckenzie.com/en/resources/global-data-and-cyber-handbook/emea/israel/topics/security-requirements-and-breach-notification) · [Pearl Cohen — PPA breach timescale + Pharma/MD regulation](https://www.pearlcohen.com/israel-privacy-protection-authority-hardens-its-policy-on-breach-notification-timescales/) · [DLA Piper — Israel cross-border transfers](https://www.dlapiperdataprotection.com/index.html?t=transfer&c=IL) · [Linklaters — Data Protected Israel](https://www.linklaters.com/en-us/insights/data-protected/data-protected---israel) · [TechPolicy.org.il — Amendment 13 brief](https://techpolicy.org.il/wp-content/uploads/2024/10/Overview-of-Amendment-no-13-FINAL-FINAL-FOR-UPLOAD-FOR-WEBSITE-COLLATED-1.pdf) · [WIPO Lex — PPL 5741-1981 text](https://www.wipo.int/wipolex/en/text/347462) · [Amendment 13 unofficial English translation](https://dpo-india.com/Resources/Privacy_Regulations_in_Middle_East/Amendment13-Israeli-Privacy-Law,5741-1981.pdf) · [Emergo by UL — AMAR registration](https://www.emergobyul.com/services/medical-device-registration-and-approval-israel) · [BIOREG — AMAR services](https://bioregservices.com/regulatory-consulting-israel-amar/) · [Eran Yona — SaMD/IVD guideline](https://eranyona.com/samd-ivd-and-e-health-services-registration-in-israel-guideline/) · [Compliance & Risks — Israel MDR guide](https://www.complianceandrisks.com/blog/a-comprehensive-guide-to-medical-device-regulation-in-israel-registration-documentation-and-labeling-requirements/) · [Hardian Health — AI device standards](https://www.hardianhealth.com/insights/regulatory-ai-medical-device-standards) · [Israel Innovation Authority — AI Health Sandbox call](https://innovationisrael.org.il/en/calls_for_proposal/ai-in-health-sandbox/) · [Cambridge Forum — Israeli AI sandbox frameworks](https://www.cambridge.org/core/journals/cambridge-forum-on-ai-law-and-governance/article/nascent-regulatory-sandbox-frameworks-for-ai-in-israel/C9E5B9630A678D809BDA109AE018881A) · [ICLG — Digital Health Israel 2026](https://iclg.com/practice-areas/digital-health-laws-and-regulations/israel) · [ICLG — Data Protection Israel 2025-26](https://iclg.com/practice-areas/data-protection-laws-and-regulations/israel) · [Censinet — Healthcare Vendor Risk Checklist 2025](https://censinet.com/perspectives/the-ultimate-healthcare-vendor-risk-assessment-checklist-for-2025) · [BigID — Amendment 13](https://bigid.com/blog/what-israel-amendment-13-means-for-businesses-in-2025/) · [INPLP — Mandatory DPO Israel](https://inplp.com/latest-news/article/new-amendment-to-israeli-privacy-protection-law-and-mandatory-dpo-appointment/)
