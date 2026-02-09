# IAM (Identity and Access Management) — Deep Dive

**Purpose:** A GitHub-ready reference that consolidates the IAM lecture/transcript content and expands the advanced layer with closely related AWS concepts and practical guidance. Follows the **Progressive Depth Model**: **Junior → Intermediate → Advanced**.

---

## Table of contents
- [Overview](#overview)  
- [Key concepts](#key-concepts)  
- [Progressive explanations (Junior → Advanced)](#progressive-explanations-junior-—-advanced)  
- [Policy evaluation: detailed mechanics](#policy-evaluation-detailed-mechanics)  
- [Policy types and governance controls](#policy-types-and-governance-controls)  
- [Practical examples and scenarios](#practical-examples-and-scenarios)  
- [Troubleshooting checklist and diagnostics](#troubleshooting-checklist-and-diagnostics)  
- [Best practices and operational guidance](#best-practices-and-operational-guidance)  
- [Simple policy examples (reference)](#simple-policy-examples-reference)  
- [References](#references)

---

## Overview
IAM is the central AWS service for identity and permission management. This document explains the basics (root user, users, groups, policies, least privilege) and then dives into how AWS evaluates policies, how different policy types interact, governance controls that can further constrain permissions, and practical troubleshooting and design guidance.

---

## Key concepts
- **Root user:** The original account identity with full privileges. Protect it with MFA and avoid daily use.  
- **IAM user:** Represents a person or service; best practice is **one user per person** for accountability.  
- **IAM group:** A container for users to simplify permission assignment; groups cannot be nested.  
- **Policies:** JSON documents that state `Effect` (`Allow` or `Deny`), `Action`, `Resource`, and optional `Condition`. Policies attach to identities (users, groups, roles) or resources.  
- **Least privilege:** Start with minimal permissions and expand only as needed.

---

## Progressive explanations (Junior — Advanced)

### Beginner
IAM controls who can do what. Create individual users, group them by role, attach policies that say what they can do, and protect the root account.

### Intermediate
Permissions come from multiple places: user policies, group policies, resource policies, and higher-level governance controls. Allows from different sources combine; an explicit Deny anywhere blocks the action. Use groups to manage common permissions and keep policies scoped.

### Advanced
Understand the full evaluation model, how governance controls (SCPs, permissions boundaries) limit effective permissions, and how resource-based policies and session policies interact. Design policies with scoped ARNs, actions, and conditions to minimize blast radius. Audit and simulate effective permissions regularly.

---

## Policy evaluation: detailed mechanics

### Core evaluation rules (concise)
1. **Implicit Deny:** By default, all actions are denied unless explicitly allowed.  
2. **Explicit Allow:** If any applicable policy explicitly allows the action on the resource, the action is permitted—*unless* an explicit Deny applies.  
3. **Explicit Deny:** If any applicable policy explicitly denies the action on the resource, the request is denied. Explicit Deny takes precedence over any Allow.

### What “all applicable policies” means
When a principal makes a request, AWS considers the union of all policy statements that could apply, including:
- Identity-based policies attached to the **user**.
- Identity-based policies attached to **groups** the user belongs to.
- **Resource-based policies** (for example, S3 bucket policies).
- **Session policies** (policies passed when assuming a role or creating temporary credentials).
- **Permissions boundaries** attached to the principal (they limit the maximum permissions the principal can obtain).
- **Service Control Policies (SCPs)** applied at the AWS Organization level (they limit what principals in member accounts can do).
- Any other applicable controls (e.g., VPC endpoint policies for some services).

### How evaluation combines statements
- **Allows are additive:** multiple Allow statements from different sources are combined (union) to permit a broader set of actions/resources.  
- **Denies are authoritative:** a single explicit Deny that matches the request (action/resource/condition) will block the request even if other statements Allow it.  
- **Conditions matter:** a statement’s `Condition` can narrow when an Allow or Deny applies (for example, by IP address, MFA presence, or request time). Only statements whose conditions evaluate to true are considered matching.

### Example evaluation flow (conceptual algorithm)
1. Collect all policy statements from identity, group(s), resource, session, SCPs, and boundaries that could apply to the request.  
2. Filter statements by whether the `Action` and `Resource` match the request and whether `Condition` (if present) evaluates to true.  
3. If any matching statement has `Effect: Deny`, the final decision is **Deny**.  
4. Else if any matching statement has `Effect: Allow`, the final decision is **Allow**.  
5. Else the final decision is **Deny** (implicit Deny).

---

## Policy types and governance controls

### Identity-based policies
- **User policies:** attached directly to a user.  
- **Group policies:** attached to a group; all members inherit them.  
- **Role policies:** attached to roles assumed by principals or services.

### Resource-based policies
- Policies attached to resources (e.g., S3 bucket policy, SQS queue policy). They can grant access to principals in other accounts and are evaluated alongside identity policies.

### Session policies
- Temporary policies passed when assuming a role or creating temporary credentials; they further restrict the permissions of the session.

### Permissions boundaries
- A permissions boundary is an identity-based policy that sets the **maximum** permissions an identity-based policy can grant. Even if an identity policy Allows an action, the permissions boundary can prevent it if the boundary does not allow it.

### Service Control Policies (SCPs)
- SCPs are applied at the AWS Organizations level and define the maximum permissions for accounts in the organization. SCPs do not grant permissions by themselves; they only limit what identity policies in member accounts can grant.

### Interaction summary
- **SCPs** and **permissions boundaries** act as upper limits; they cannot grant permissions but can restrict them.  
- **Resource-based policies** can grant access to principals outside the account and are evaluated in combination with identity policies.  
- **Explicit Deny** in any of these policy types will block the action.

---

## Practical examples and scenarios

### Scenario 1 — Group Allow vs User Deny (your original example)
- **Group policy:** `Allow` read-only (`s3:GetObject`, `s3:ListBucket`) on `arn:aws:s3:::example-bucket/*`.  
- **User policy:** `Deny` `s3:*` on `arn:aws:s3:::example-bucket/*`.  
- **Effective result:** **Denied**. The explicit Deny in the user policy overrides the group Allow.

### Scenario 2 — Resource policy grants cross-account access but SCP denies it
- **Bucket policy:** Allows `s3:GetObject` for a principal in a member account.  
- **SCP:** Denies `s3:GetObject` for that member account.  
- **Effective result:** **Denied**. SCP prevents the account from performing the action even though the resource policy allows it.

### Scenario 3 — Permissions boundary restricts a user
- **User policy:** Allows `ec2:StartInstances`.  
- **Permissions boundary:** Does not include `ec2:StartInstances`.  
- **Effective result:** **Denied**. The boundary prevents the user from obtaining that permission.

### Scenario 4 — Conditions narrow a Deny
- **User policy:** Allows `s3:GetObject` on a bucket.  
- **Group policy:** Denies `s3:GetObject` on the bucket unless `aws:MultiFactorAuthPresent` is `true`.  
- **If** the user’s session has MFA present → Deny condition evaluates false → Allow applies.  
- **If** no MFA → Deny applies → request denied.

---

## Troubleshooting checklist and diagnostics

When a user is unexpectedly denied access:

1. **Confirm the exact API call or console action** and the resource ARN being accessed.  
2. **Collect all applicable policies**:
   - User policies (inline and managed).  
   - Group policies for all groups the user belongs to.  
   - Resource-based policies (e.g., S3 bucket policy).  
   - Session policies (if the user assumed a role).  
   - Permissions boundaries attached to the user or role.  
   - SCPs applied to the account via AWS Organizations.  
3. **Look for explicit Deny statements** that match the action/resource/condition. If any exist, they explain the denial.  
4. **Check conditions** in matching statements (IP, MFA, time, VPC endpoint, etc.). A condition mismatch can make an Allow not apply.  
5. **Simulate the policy** (AWS provides simulation tools) to see the effective decision for a given principal and action.  
6. **Audit CloudTrail** for the denied request to see the exact request parameters and error message.  
7. **Review group memberships** to ensure the user isn’t inheriting unexpected permissions.  
8. **Check for session caching**: long-lived sessions or assumed-role sessions may persist permissions until they expire.

**Diagnostic tips (conceptual):**
- Use policy simulation to reproduce the decision.  
- Narrow the scope of Deny statements to avoid accidental blocking.  
- When adding Deny guardrails, document the reason and scope.

---

## Best practices and operational guidance

- **Protect the root account:** enable MFA, store credentials securely, and avoid daily use.  
- **One user per person:** ensures accountability and easier revocation.  
- **Use groups and roles:** groups for human users, roles for services and cross-account access.  
- **Prefer managed policies:** AWS-managed or customer-managed policies are easier to audit and reuse than inline policies.  
- **Apply least privilege:** start with minimal permissions and expand only when necessary.  
- **Use conditions:** tighten policies with conditions (MFA, source IP, VPC endpoint, time).  
- **Use explicit Deny for guardrails:** but scope Deny statements narrowly and document them.  
- **Governance:** use SCPs and permissions boundaries to enforce organizational guardrails.  
- **Audit regularly:** review IAM Access Analyzer, CloudTrail logs, and policy simulations to detect over-privileged identities.  
- **Automate checks:** integrate IAM policy checks into CI/CD pipelines and infrastructure-as-code validations.

---

## Simple policy examples (reference)

**EC2 describe allow**
```json
{
  "Version":"2012-10-17",
  "Statement":[{"Effect":"Allow","Action":"ec2:Describe*","Resource":"*"}]
}
```

**S3 read-only for a bucket**
```json
{
  "Version":"2012-10-17",
  "Statement":[{"Effect":"Allow","Action":["s3:GetObject","s3:ListBucket"],"Resource":["arn:aws:s3:::example-bucket","arn:aws:s3:::example-bucket/*"]}]
}
```

**Scoped explicit Deny for a bucket**
```json
{
  "Version":"2012-10-17",
  "Statement":[{"Effect":"Deny","Action":"s3:*","Resource":["arn:aws:s3:::example-bucket","arn:aws:s3:::example-bucket/*"]}]
}
```

---

## Exam tips by exam

### SAA-C03
**Focus areas for IAM-related questions**
- Recognize when to use **users vs roles vs groups**; prefer roles for cross-account or service access.  
- Understand that **IAM is global** and root user must be protected.  
- Know the **least privilege** principle and how to scope policies by action and resource.  
- Identify common architecture patterns that minimize operational overhead (use managed services and roles).  

**How to approach IAM questions**
- Look for answers that **reduce operational burden** (managed policies, roles, automation).  
- Prefer solutions that **limit blast radius** (scoped ARNs, conditions).  
- Watch for distractors that propose sharing root credentials or using broad `*` policies.

**Common traps**
- Confusing nested groups (they do not exist).  
- Choosing user credentials for cross-account access instead of roles.  
- Overlooking SCPs or permissions boundaries in multi-account scenarios.

**Study actions**
- Practice mapping requirements to identity constructs (user, group, role).  
- Review examples of scoped policies and role assumption flows.

---

### DVA-C03
**Focus areas for IAM-related questions**
- Programmatic access patterns: API keys, SDK usage, and temporary credentials via roles.  
- How policies affect SDK/API calls and the importance of least privilege for applications.  
- Session policies and temporary credentials behavior.

**How to approach IAM questions**
- Prefer solutions that use **roles and temporary credentials** for applications and CI/CD.  
- Look for answers that **limit credentials exposure** (avoid long-lived access keys when possible).  
- Consider retry/backoff and error handling when permissions are insufficient.

**Common traps**
- Granting broad programmatic credentials to applications instead of scoped roles.  
- Assuming user policies alone control access without considering resource policies or boundaries.  
- Forgetting to secure access keys and rotate them.

**Study actions**
- Practice creating roles for EC2, Lambda, and CI/CD pipelines and understand how SDKs obtain temporary credentials.  
- Review how to scope S3 and DynamoDB permissions for application use cases.

---

### SAP-C02
**Focus areas for IAM-related questions**
- Governance at scale: SCPs, permissions boundaries, multi-account strategies, and least-privilege enforcement.  
- Trade-offs between centralized control (SCPs, centralized IAM) and developer agility.  
- Designing incremental migration paths and operationally sustainable guardrails.

**How to approach IAM questions**
- Think in **phases**: what is the least disruptive, most governable path forward?  
- Favor solutions that **minimize operational overhead** while meeting business constraints (use managed services, automation, and guardrails).  
- Evaluate cost, risk, and operability trade-offs explicitly.

**Common traps**
- Overengineering with overly complex custom controls when AWS-managed guardrails suffice.  
- Ignoring permissions boundaries or SCPs when designing cross-account access.  
- Choosing technically correct but operationally heavy solutions.

**Study actions**
- Practice designing multi-account architectures with SCPs and permissions boundaries.  
- Prepare to justify choices based on operational burden, governance, and business constraints.

---

## References

- **Lecture video & transcript (primary source):** Udemy lecture — IAM (current tab)  
  `https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/learn/lecture/26100726`  

- **AWS IAM policy evaluation logic (official):** Policy evaluation logic — AWS Identity and Access Management  
  `https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html`

---

