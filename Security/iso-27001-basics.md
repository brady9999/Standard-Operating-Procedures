# ISO 27001 Basics
> An introduction to the ISO 27001 information security management standard — what it is, why it matters, and how it works.

**Category:** Security  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **ISO** | International Organization for Standardization | The international body that publishes standards |
| **ISO 27001** | ISO/IEC 27001 | The international standard for Information Security Management Systems |
| **ISMS** | Information Security Management System | The framework of policies, processes, and controls for managing information security |
| **Annex A** | Annex A Controls | A list of 93 security controls in ISO 27001:2022 that organizations can implement |
| **Risk Assessment** | Risk Assessment | The process of identifying, analyzing, and evaluating information security risks |
| **Risk Treatment** | Risk Treatment | Deciding how to respond to identified risks — accept, mitigate, transfer, or avoid |
| **SoA** | Statement of Applicability | A document listing which Annex A controls apply to the organization and why |
| **Audit** | Security Audit | A systematic review to verify that the ISMS is working as intended |
| **Certification Body** | Certification Body | An accredited third-party organization that audits and certifies compliance |
| **Clause** | Clause | The main sections of the ISO 27001 standard (Clauses 4-10) |
| **Control** | Security Control | A measure put in place to mitigate a security risk |
| **Nonconformity** | Nonconformity | A failure to meet a requirement of the standard |
| **PDCA** | Plan-Do-Check-Act | The continuous improvement cycle used in ISO standards |
| **CIA** | Confidentiality, Integrity, Availability | The three core security objectives ISO 27001 protects |
| **Asset** | Information Asset | Anything of value that needs protection — data, systems, people, processes |
| **Threat** | Threat | Something that could cause harm to an information asset |
| **Vulnerability** | Vulnerability | A weakness that a threat could exploit |
| **Incident** | Security Incident | An event that has compromised or threatens to compromise information security |

---

## Overview — What is ISO 27001?

ISO 27001 is the world's leading international standard for Information Security Management. It provides a framework for establishing, implementing, maintaining, and continually improving an Information Security Management System (ISMS).

**In simple terms:** ISO 27001 tells you HOW to manage information security systematically — not just which technical controls to use, but how to identify what you need to protect, assess the risks, implement appropriate controls, and keep improving.

**Why it matters:**
- Demonstrates to customers and partners that you take security seriously
- Required by many government and enterprise contracts
- Reduces the likelihood and impact of security incidents
- Provides a structured approach to managing security rather than ad-hoc fixes
- Helps meet regulatory requirements (GDPR, PIPEDA, etc.)

---

## The ISO 27001 Structure

### Main Clauses (Mandatory — must all be implemented)

| Clause | Title | What It Covers |
|--------|-------|----------------|
| **4** | Context of the Organization | Understanding the organization, stakeholders, and ISMS scope |
| **5** | Leadership | Management commitment, security policy, roles and responsibilities |
| **6** | Planning | Risk assessment, risk treatment, security objectives |
| **7** | Support | Resources, competence, awareness, communication, documentation |
| **8** | Operation | Implementing and controlling security processes |
| **9** | Performance Evaluation | Monitoring, measurement, audits, management review |
| **10** | Improvement | Handling nonconformities, continual improvement |

### Annex A — Security Controls (Select applicable ones)

ISO 27001:2022 has **93 controls** organized into **4 themes:**

| Theme | Controls | Examples |
|-------|----------|---------|
| **Organizational** | 37 controls | Policies, roles, supplier relationships, incident management |
| **People** | 8 controls | Screening, training, disciplinary process, remote working |
| **Physical** | 14 controls | Physical access, equipment security, clear desk policy |
| **Technological** | 34 controls | Access control, encryption, malware protection, logging |

---

## The PDCA Cycle — How ISO 27001 Works

ISO 27001 is based on continual improvement using the Plan-Do-Check-Act cycle:

```
PLAN
├── Identify information assets
├── Assess risks to those assets
├── Select appropriate controls
└── Set security objectives

DO
├── Implement the selected controls
├── Train staff on security procedures
└── Operate the ISMS

CHECK
├── Monitor effectiveness of controls
├── Conduct internal audits
├── Review security incidents
└── Management review

ACT
├── Take corrective action on failures
├── Improve controls based on findings
└── Update risk assessment as things change
```

---

## 1. The Risk Assessment Process

Risk assessment is the foundation of ISO 27001 — everything else flows from it.

### Step 1 — Identify Assets
List everything that needs to be protected:
```
Information assets:
- Customer data, financial records, employee data
- Source code, configuration files, documentation

Systems:
- Servers, databases, applications, network equipment

People:
- Employees, contractors, key personnel

Physical:
- Server rooms, offices, equipment
```

### Step 2 — Identify Threats and Vulnerabilities

For each asset, identify:
- **Threats** — What could go wrong? (fire, malware, theft, human error, power failure)
- **Vulnerabilities** — What weaknesses could be exploited? (unpatched systems, weak passwords, no backups)

### Step 3 — Assess Risk

```
Risk = Likelihood × Impact

Likelihood: How likely is this threat to occur?
  1 = Rare
  2 = Unlikely
  3 = Possible
  4 = Likely
  5 = Almost certain

Impact: How severe would the consequences be?
  1 = Negligible
  2 = Minor
  3 = Moderate
  4 = Major
  5 = Catastrophic

Risk Score: 1-8 = Low, 9-14 = Medium, 15-25 = High
```

### Step 4 — Risk Treatment

For each risk decide how to respond:

| Option | What It Means | Example |
|--------|---------------|---------|
| **Mitigate** | Implement controls to reduce the risk | Install antivirus, enable MFA |
| **Accept** | Accept the risk as-is (low risk only) | Accepting minor inconvenience risks |
| **Transfer** | Transfer the risk to a third party | Cyber insurance |
| **Avoid** | Stop the activity that creates the risk | Stop using a vulnerable system |

---

## 2. Key Security Controls (Annex A Examples)

### Access Control
- All users have unique accounts — no shared logins
- Access is granted on a need-to-know basis (least privilege)
- Privileged access (admin accounts) is controlled and reviewed regularly
- Access is revoked immediately when employment ends

### Cryptography
- Sensitive data is encrypted at rest and in transit
- Encryption keys are managed securely
- Strong algorithms are used (AES-256, SHA-256, TLS 1.2+)

### Physical Security
- Server rooms and data centres have restricted access
- Equipment is protected from environmental hazards
- Clear desk and clear screen policies are enforced
- Visitors are escorted

### Operations Security
- Systems are patched regularly
- Malware protection is installed and updated
- Backups are taken, tested, and stored securely
- System activity is logged and reviewed

### Incident Management
- A procedure exists for reporting and responding to security incidents
- Incidents are documented, analyzed, and learned from
- Staff know how to report a suspected incident

### Supplier Relationships
- Third-party vendors who handle your data are assessed for security
- Contracts include security requirements
- Third-party access is controlled and monitored

---

## 3. The Statement of Applicability (SoA)

The SoA is one of the most important documents in your ISMS. It lists:
- Which of the 93 Annex A controls you have selected
- Which controls you have excluded and why
- The current implementation status of each control

**Example SoA row:**

| Control | Applicable | Justification | Status |
|---------|-----------|---------------|--------|
| A.8.1 User endpoint devices | Yes | Staff use laptops and mobile devices | Implemented |
| A.8.20 Networks security | Yes | Company operates a network | Implemented |
| A.5.14 Information transfer | Yes | Data is shared with clients | In progress |
| A.7.4 Physical security monitoring | No | No physical premises requiring cameras | Excluded |

---

## 4. Documentation Requirements

ISO 27001 requires specific documented information. You must be able to produce these:

**Mandatory documents:**
- Information security policy
- Risk assessment methodology and results
- Risk treatment plan
- Statement of Applicability
- Information security objectives
- Competence records (evidence of training)
- Operational planning and control evidence
- Results of monitoring and measurement
- Internal audit results
- Results of management review
- Nonconformities and corrective actions

**Recommended but not mandatory:**
- Asset inventory
- Acceptable use policy
- Access control policy
- Business continuity plan
- Incident response procedure
- Supplier security policy

---

## 5. Internal Audits

ISO 27001 requires regular internal audits to verify the ISMS is working.

**What auditors check:**
- Are documented policies actually being followed?
- Are controls implemented as described?
- Is there evidence of monitoring and review?
- Are incidents being recorded and addressed?
- Is the risk assessment current?

**Audit process:**
```
1. Plan — determine scope, schedule, and criteria
2. Conduct — review documentation, interview staff, observe processes
3. Report — document findings, nonconformities, and observations
4. Follow up — verify corrective actions are completed
```

---

## 6. ISO 27001 Certification Process

Getting certified involves:

```
Stage 1 — Gap Analysis
  Assess current state against ISO 27001 requirements
  Identify what needs to be implemented

Stage 2 — Implementation
  Build the ISMS
  Document policies and procedures
  Implement technical and organizational controls
  Train staff

Stage 3 — Internal Audit
  Audit your own ISMS before the external audit
  Fix any gaps found

Stage 4 — Management Review
  Senior management reviews the ISMS status

Stage 5 — Stage 1 Audit (Documentation Review)
  External certification body reviews your documentation
  Confirms the ISMS is ready for the next stage

Stage 6 — Stage 2 Audit (Certification Audit)
  Certification body audits the actual implementation
  Interviews staff, reviews evidence

Stage 7 — Certification Issued
  Certificate valid for 3 years
  Annual surveillance audits to maintain certification

Stage 8 — Continual Improvement
  Ongoing monitoring, audits, and improvement
```

---

## 7. ISO 27001 in Day-to-Day IT Work

As an IT professional supporting an organization working toward ISO 27001 you would be involved in:

**Access Control:**
- Ensuring user accounts follow least privilege
- Reviewing and revoking access promptly when staff leave
- Managing privileged access accounts

**Logging and Monitoring:**
- Ensuring security logs are enabled and retained
- Reviewing logs for suspicious activity
- Responding to security alerts

**Patch Management:**
- Keeping systems patched and up to date
- Documenting patch activities
- Prioritizing critical vulnerabilities

**Incident Response:**
- Detecting and reporting security incidents
- Following the incident response procedure
- Documenting what happened and how it was resolved

**Documentation:**
- Keeping system documentation current
- Recording configuration changes
- Maintaining asset inventories

**Supplier Management:**
- Controlling third-party access to systems
- Ensuring vendors meet security requirements

---

## 8. Common ISO 27001 Nonconformities

These are the most common failures found during audits:

| Nonconformity | Why It Happens | How to Fix |
|---------------|----------------|------------|
| Incomplete risk assessment | Risk register not updated | Review and update at least annually |
| No evidence of training | Training done but not documented | Record all security awareness training |
| Stale access permissions | No offboarding process | Implement formal access review and revocation |
| Missing incident records | Incidents handled informally | Implement incident logging — even minor ones |
| Outdated documentation | Procedures written but never maintained | Schedule regular document reviews |
| No management review | Leadership not engaged | Schedule formal quarterly management reviews |
| SoA not reflecting reality | Controls documented but not implemented | Audit controls against SoA regularly |

---

## Quick Reference — Key Concepts

| Concept | Key Point |
|---------|-----------|
| **ISMS** | A framework, not just a technical solution — covers people, processes, and technology |
| **Risk-based approach** | Controls are selected based on risk — not every control applies to every organization |
| **Continual improvement** | ISO 27001 is never "done" — it requires ongoing review and improvement |
| **Evidence** | Auditors need evidence — document everything |
| **Scope** | You define what is in scope — not everything has to be included initially |
| **Top management** | Leadership must be visibly committed — cannot be delegated entirely to IT |
| **Annex A** | The controls are a menu — pick what applies, document why you excluded others |

---

## Notes
- ISO 27001 is a management standard — it covers governance, not just technology
- Certification is valuable but not mandatory — you can implement the standard without getting certified
- ISO 27001:2022 is the current version — updated from 2013 with reorganized controls
- The standard works alongside other frameworks — NIST CSF, SOC 2, GDPR compliance


---

## Related Documents
- [Network Security Basics](network-security-basics.md)
- [Active Directory Password & Group Policy](../Windows/active-directory-password-local--group-policy.md)
- [Windows Event Viewer](../Windows/windows-event-viewer.md)
