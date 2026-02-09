# Overview

This document consolidates our accumulated IAM study notes into a single, GitHub‑ready reference. It covers identities (users, groups, roles), policies, access keys, MFA, security tools, hands‑on examples, operational best practices, and exam‑focused guidance for SAA, DVA, and SAP levels. Use this as a living reference: it is organized to start simple and progressively deepen into technical and operational detail.

---

## Core Concepts

### Identity Types
- **Root account**  
  **Definition:** The account owner identity with full control.  
  **Rule:** Reserve for account setup and emergencies; enable MFA immediately.

- **IAM User**  
  **Definition:** A permanent identity for a person or service that needs long‑term credentials.  
  **Usage:** Human users for console access (password + MFA) or programmatic access (access keys) when federation is not possible.

- **Group**  
  **Definition:** A collection of users that inherit attached policies.  
  **Usage:** Role‑based grouping (Developers, Operators, Auditors) to simplify permission management.

- **Role**  
  **Definition:** An assumable identity that issues temporary credentials (via STS) to services or principals.  
  **Usage:** EC2 instance profiles, Lambda execution roles, cross‑account access, federated identities.

### Policies and Policy Types
- **Identity‑based policies**: Attached to users, groups, or roles.  
- **Resource‑based policies**: Attached to resources (S3 bucket policy, SQS, SNS, KMS key policy).  
- **Managed policies**: AWS‑managed or customer‑managed (reusable).  
- **Inline policies**: Embedded directly on a single identity (not reusable).  
- **Policy structure**: `Version`, `Statement` → `Sid`, `Effect` (`Allow`/`Deny`), `Action`, `Resource`, `Condition`.

### Policy Evaluation Summary
- **Default Deny**: Nothing is allowed unless explicitly allowed.  
- **Explicit Deny**: Overrides any Allow.  
- **Effective permissions**: Union of applicable allows across user, group, role, resource, session policies, intersected with permission boundaries and SCPs.  
- **Order of influence**: SCPs and permission boundaries act as caps; explicit Deny anywhere blocks access.

### Temporary Credentials and STS
- **STS flows**: `AssumeRole`, `GetSessionToken`, `GetFederationToken`.  
- **Benefits**: Short‑lived credentials, reduced blast radius, no long‑lived secrets in code.

---

## Hands‑On Labs and Examples

### Lab 1 — Create User, Group, Attach Policy, Validate
**Goal:** Create a user, add to a group with a managed policy, validate permissions.

**Steps (console or CLI):**
1. Create group and attach policy:
```bash
aws iam create-group --group-name Developers
aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/IAMReadOnlyAccess
```
2. Create user and add to group:
```bash
aws iam create-user --user-name alice
aws iam add-user-to-group --user-name alice --group-name Developers
```
3. Validate from CLI (after configuring credentials for alice):
```bash
aws sts get-caller-identity
aws iam list-users
```

---

### Lab 2 — Role for EC2 and Instance Profile
**Goal:** Create a service role for EC2, attach minimal permissions, launch instance with instance profile.

**CLI sequence:**
```bash
# Create role with trust policy for EC2 (trust-policy.json)
aws iam create-role --role-name EC2AppRole --assume-role-policy-document file://trust-policy.json

# Attach permission policy
aws iam attach-role-policy --role-name EC2AppRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile and add role
aws iam create-instance-profile --instance-profile-name EC2AppProfile
aws iam add-role-to-instance-profile --instance-profile-name EC2AppProfile --role-name EC2AppRole
```
**Validation:** Launch EC2 with the instance profile and verify access via instance metadata credentials.

---

### Lab 3 — MFA Enablement and CLI with MFA
**Goal:** Enable virtual MFA for a user and obtain temporary credentials.

**Enable via CLI (requires two consecutive codes):**
```bash
aws iam enable-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice \
  --authentication-code1 123456 \
  --authentication-code2 789012
```
**Get temporary credentials with MFA:**
```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/alice \
  --token-code 123456
```

---

### Lab 4 — Policy Authoring Example
**Least‑privilege S3 read policy**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyS3Objects",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket",
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

---

## Best Practices and Operational Guidance

### Identity and Credential Hygiene
- **Root account**: Enable MFA; do not use for daily tasks.  
- **One user per person**: Avoid shared accounts.  
- **Groups over inline user policies**: Attach policies to groups for reuse and auditability.  
- **Prefer roles and STS**: For services, automation, and cross‑account access.  
- **Access keys**: Create only when necessary; store secret keys securely; rotate and remove unused keys.

### MFA Strategy
- **Minimum**: MFA on root and all privileged users.  
- **Preferred**: Hardware FIDO2 security keys for administrators (phishing resistant).  
- **Virtual MFA**: Acceptable for general users; plan backups and recovery.  
- **Enforcement**: Use IAM policy conditions (e.g., `aws:MultiFactorAuthPresent`) or SCPs to require MFA for sensitive actions.

### Policy Engineering
- **Least privilege first**: Start narrow and expand only as required.  
- **Avoid wildcards**: Do not use `Action: "*"` or `Resource: "*"` unless justified.  
- **Use Conditions**: Restrict by `aws:SourceIp`, `aws:SourceVpc`, `aws:MultiFactorAuthPresent`, `aws:SourceAccount`, `aws:SourceArn`.  
- **Permission boundaries**: Use to cap delegated admin privileges.  
- **SCPs**: Use in Organizations to enforce org‑wide guardrails.

### Security Tools and Automation
- **IAM Credentials Report**: Periodic snapshot of credential health (passwords, access keys, MFA).  
- **IAM Access Analyzer**: Detects external access, unused permissions, and can generate least‑privilege policy suggestions.  
- **CloudTrail**: Audit STS, AssumeRole, sign‑in, and API calls. Correlate events for anomaly detection.  
- **Automate rotation**: Use Secrets Manager + Lambda/CI pipelines to rotate keys and update applications.  
- **Centralized governance**: Aggregate reports and logs from member accounts; automate stale credential detection and remediation.

### Operational Tradeoffs
- **Hardware MFA**: Stronger security, higher operational overhead (inventory, issuance, replacement).  
- **Virtual MFA**: Lower friction, easier onboarding, but less phishing resistant.  
- **Long‑lived keys vs roles**: Long‑lived keys are operationally simpler but increase risk; roles + STS are safer and recommended.

---

## Exam Tips by Certification

### SAA‑C03 Solutions Architect Associate
- **Focus**: Correct service selection, availability, durability, scalability.  
- **IAM emphasis**: Prefer managed services, use roles for services, enforce least privilege.  
- **Common traps**: Choosing root or long‑lived keys for automation; confusing resource vs identity policies.

### DVA‑C03 Developer Associate
- **Focus**: SDKs, APIs, CI/CD, serverless, retries, and error handling.  
- **IAM emphasis**: Use roles for CI/CD and Lambda; prefer temporary credentials; understand `aws sts assume-role` flows.  
- **Common traps**: Using user access keys in code; not using session tokens for temporary access.

### SAP‑C02 Solutions Architect Professional
- **Focus**: Trade‑offs, governance, migration, cost vs resilience.  
- **IAM emphasis**: Multi‑account strategies (Organizations, SCPs), permission boundaries, delegated admin models.  
- **Exam mindset**: Multiple technically correct answers exist — choose the solution with least operational burden, managed services, and incremental migration paths.

---

# References

**Primary learning sources saved to study notes**
- Course transcript and hands‑on labs from the current Udemy IAM lecture.  
- AWS Identity and Access Management documentation and best practices.  
- AWS Security and Compliance guidance for MFA, STS, and key rotation.  
- IAM Access Analyzer documentation and least‑privilege policy generation guidance.  
- Community and AWS prescriptive guidance on access key rotation and automation.

---
