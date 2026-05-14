# R4 — Israeli cloud deployment infrastructure

Architecture target:
- Web/mobile frontend
- Backend API in Israeli-region cloud
- Patient audio + transcripts + clinical features stored in Israeli region
- STT calls go OUT to Soniox EU region (DPA + patient consent)
- Optional LLM inference (DictaLM 1.7B-Base) for narrow Hebrew post-processing

Compliance: PPL Amendment 13, HIPAA-equivalent posture, audit-ready.

---

## 1. AWS Tel Aviv (il-central-1)

- **GA since August 1, 2023**. Three AZs.
- AWS states "all AWS services" available; community catalog lists ~134 services.
- SageMaker is GA.
- **Amazon Bedrock supported in il-central-1, but Anthropic Claude models are accessible only via Geo (Sonnet 4) or Global cross-region routing — no in-region Claude inference.** Inference egresses the Israeli region.
- EC2, RDS (Postgres), S3, Lambda, CloudWatch, KMS, Cognito, Secrets Manager, ECS/EKS, ELB, Route 53 all confirmed for any new region.
- **Pricing premium**: ~5–6% over Ireland on EC2; data egress from Israel is **~4× costlier** than US/EU egress per DoiT analysis.
- Premium vs us-east-1 not stated in a single source; assume 10–15% blended.

## 2. Azure Israel Central

- Launched 2023 (quietly). **3 availability zones**.
- Region complies with data-at-rest residency in Israel.
- **HIPAA BAA covers in-scope Azure services globally** under Microsoft Product Terms — region does not gate BAA, but Microsoft does not publish a per-region HIPAA-eligible matrix.
- Service catalog narrower than AWS in early years; verify each service before commit.

## 3. Oracle Cloud Jerusalem (il-jerusalem-1)

- **GA October 11, 2021** — first hyperscaler in Israel.
- 65+ services including Autonomous Database, OKE (Kubernetes), Oracle Cloud VMware.
- Underground bunker facility in Har Hotzvim.
- **Single region in Israel; no second OCI Israeli region for cross-region DR within country.**

## 4. GCP Israel (me-west1)

- **GA October 2022, Tel Aviv.** Part of Project Nimbus.
- For non-GCP latency reference: Israel ↔ Belgium (europe-west1) fiber path is roughly 3,300 km — speed-of-light floor ~33ms RTT; real-world typically 60–80ms.

## 5. Israeli sovereign cloud

- **Bynet Cloud** — markets itself as Israel's "first sovereign cloud" — Israeli entity, Israeli law, suitable for private/public healthcare.
- **Triple-C** — runs Israel's largest local public cloud, serving banks, insurance, government, health.
- **Project Nimbus** — $1.2B government contract with AWS+Google to host **Israeli government** workloads with anti-boycott clauses. **Not a service open to private healthcare apps** — private apps just use the standard commercial il-central-1 / me-west1 regions on commercial terms.

## 6. Israel ↔ Soniox EU latency

- Soniox runs EU endpoints (Frankfurt area typical for STT vendors).
- Tel Aviv ↔ Frankfurt distance ~3,000 km → ~30ms fiber RTT floor.
- Widely reported real-world AWS inter-region RTT for this corridor: **50–70ms range**.
- Verify with [cloudping.co](https://www.cloudping.co/) before architectural commit.
- Cumulative latency budget for round-trip STT: ~70ms network + STT processing.

## 7. HIPAA eligibility

AWS's HIPAA Eligible Services list (100+ services) includes:
- EC2, RDS, S3, Lambda, CloudWatch, KMS, Cognito, Secrets Manager
- ECS, EKS, ELB, Route 53
- SageMaker, Bedrock, Transcribe, Comprehend Medical

Eligibility is service-level, not region-gated, once a BAA is signed.

**Note:** PPL Amendment 13 (in force Aug 14, 2025) is the binding Israeli regime; HIPAA is a US standard used here only as a security posture analogue. Amendment 13 mandates encryption, access logs, pen-tests, DPO, and an adequacy/contract regime for cross-border transfer — Soniox EU transfer needs a documented DPA + patient consent.

## 8. Cost estimate (rough, unverified — use AWS Calculator before commit)

Workload of 12,000 hr/month audio + transcripts in il-central-1:

| Component | Estimate |
|---|---|
| 3× t3/m6i medium EC2 + ALB | ~$300–500 |
| RDS Postgres db.t3.medium Multi-AZ | ~$200 |
| S3 Standard 1TB + requests | ~$30 |
| KMS + Secrets Manager | ~$10 |
| CloudWatch logs (audit-grade) | ~$50–150 |
| **Egress to Soniox EU (audio ~12 TB @ ~$0.08/GB Israel-out)** | **~$960** (dominant) |
| **Total** | **~$1,600–1,900/mo** |

Same workload us-east-1: ~$1,100–1,300/mo (egress at ~$0.02/GB → ~$240).

Egress to Soniox dominates >50% of cost.

## 9. Data residency contractual specifics

- AWS contractually commits that **customer content is not moved out of the chosen Region** for in-region services like S3 Standard, RDS, EBS.
- **Global services replicate by design** — DynamoDB Global Tables, CloudFront, Route 53, IAM — exclude these from PHI flows or pin to single-region equivalents.

## 10. Disaster recovery

- **AWS has only il-central-1 in Israel** — no in-country secondary.
- For residency-preserving DR you must either:
  - (a) accept single-region with multi-AZ + backup snapshots stored in il-central-1, or
  - (b) cross-region DR to eu-central-1 (Frankfurt) under Amendment 13 cross-border transfer documentation.
- **Oracle and Azure also single-Israel-region. No hyperscaler offers in-country cross-region DR in Israel.**

## 11. Pen-test partners (Israel)

- **Sygnia** (Tel Aviv, healthcare/AI experience).
- **Comsec** (SOC 2, healthcare pen tests confirmed in customer references).
- **Cymotive is automotive-only — not relevant.**
- Other candidates worth evaluating: Check Point Professional Services, CYE, Cynet.

---

## Recommended primary stack

**AWS il-central-1** as primary:
- Largest in-region service catalog including SageMaker for in-region DictaLM hosting.
- Three AZs.
- Signed BAA covers EC2/RDS/S3/KMS/Cognito/Lambda/Secrets Manager.
- Strongest Israeli-press-confirmed enterprise adoption.

Run Soniox STT calls over a documented Amendment 13 cross-border transfer to Soniox EU (Frankfurt-region) — accept the ~60ms RTT and the ~4× egress premium as the cost of residency.

DR to eu-central-1 under explicit transfer documentation.

Engage **Sygnia or Comsec** for the mandatory Amendment-13 pen-test on the audio+PHI database.

**Avoid Bedrock for in-region Claude inference** (it leaves Israel via Global routing) — host DictaLM on SageMaker in il-central-1 instead.

## Caveats

- (a) Tel Aviv↔Frankfurt RTT cited as 50–70ms range from speed-of-light + community measurements — confirm with cloudping.co before architecture lock.
- (b) The $1,600–1,900/mo cost is an unrun estimate — run AWS Pricing Calculator before commit.
- (c) Premium vs us-east-1 stated as 10–15% blended — no single source confirms this exact figure; only the 5–6% vs Ireland comparison is grounded.
- (d) Bedrock Claude in-region routing claim verified from AWS Bedrock model-region-compatibility docs as of search date.

## Sources

[Now Open – AWS Israel (Tel Aviv) Region — AWS Blog](https://aws.amazon.com/blogs/aws/now-open-aws-israel-tel-aviv-region/) · [AWS Israel (Tel Aviv) Region](https://aws.amazon.com/local/israel/) · [AWS Services in il-central-1 (catalog)](https://awsfundamentals.com/regions/il-central-1) · [Amazon Bedrock Regional Availability](https://docs.aws.amazon.com/bedrock/latest/userguide/models-region-compatibility.html) · [AWS HIPAA Eligible Services Reference](https://aws.amazon.com/compliance/hipaa-eligible-services-reference/) · [DoiT — AWS Tel Aviv region price comparison](https://www.doit.com/blog/aws-region-in-tel-aviv-israel-price-comparison-versus-other-regions/) · [Microsoft quietly launches Israeli Azure cloud region — DCD](https://www.datacenterdynamics.com/en/news/microsoft-quietly-launches-israeli-azure-cloud-region/) · [Azure Israel Central — azurespeed](https://www.azurespeed.com/Information/AzureRegions/IsraelCentral) · [Azure HIPAA Compliance offering — Microsoft Learn](https://learn.microsoft.com/en-us/azure/compliance/offerings/offering-hipaa-us) · [Oracle Israel Cloud Regional Launch](https://www.oracle.com/il-en/cloud/cloud-regions/israel/) · [Oracle opens Israeli Cloud region in Jerusalem — DCD](https://www.datacenterdynamics.com/en/news/oracle-opens-israeli-cloud-region-in-jerusalem/) · [Google Cloud region in Tel Aviv now open — Google Cloud Blog](https://cloud.google.com/blog/products/infrastructure/new-google-cloud-region-in-israel-is-now-open) · [Project Nimbus — Wikipedia](https://en.wikipedia.org/wiki/Project_Nimbus) · [Bynet Cloud — sovereign IL cloud](https://www.bynet.co.il/en/solutions/bynet-cloud-il/) · [Triple-C — Petah Tikva data center](https://www.datacentermap.com/israel/petah-tikva/triplec-petah-tikva/) · [Israel Privacy Law Amendment 13 — IAPP](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform) · [Amendment 13 brief — Tech Policy Israel](https://techpolicy.org.il/wp-content/uploads/2024/10/Overview-of-Amendment-no-13-FINAL-FINAL-FOR-UPLOAD-FOR-WEBSITE-COLLATED-1.pdf) · [AWS Region Latency Matrix — cloudping.co](https://www.cloudping.co/) · [Sygnia (Tel Aviv, AI/healthcare cyber)](https://www.sygnia.co/) · [Comsec — penetration testing services](https://com-sec.io/penetration-testing-services)
