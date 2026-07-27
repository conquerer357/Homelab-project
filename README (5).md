# Task 4: Help Desk Scenarios Logged in ServiceNow

## Objective

I ran two common service desk scenarios end to end in my lab domain and logged each one as a ticket in a ServiceNow Personal Developer Instance. The goal was to practise the ticket handling side of an IT support role, not just the Active Directory side, so I treated the ticket quality as part of the deliverable rather than an afterthought.

The two scenarios were an account lockout and a new starter account setup.

Environment: Windows Server 2022 domain controller (`akvlab.local`) and a domain joined Windows 11 client, both on a VirtualBox internal network. Builds on the shared folder and permission model from Task 3.

## Steps Taken

### Scenario 1: Account lockout

1. Configured an account lockout policy in the Default Domain Policy, which is linked at the domain root. Threshold 5 invalid attempts, lockout duration 15 minutes, observation window 15 minutes. Verified the applied values with `Get-ADDefaultDomainPasswordPolicy` rather than reading them back from the Group Policy editor.
2. Triggered a real lockout by attempting six failed logons as `jdoe` from the client, and captured the lockout message shown at the logon screen.
3. Unlocked the account in Active Directory Users and Computers via the Account tab, then confirmed a successful logon from the client.
4. Logged the work as incident INC0010001 with caller, category, impact and urgency, work notes, resolution code and customer facing resolution notes.

### Scenario 2: New starter account setup

1. Created a domain user account in the Finance OU following the same first initial plus surname naming convention already used for the other Finance accounts, and set User must change password at next logon so the starter sets their own password.
2. Added the account to `SG_Finance_Read` rather than `SG_Finance_Modify`, matching the access level requested and keeping to least privilege.
3. Verified from the client after a full sign out and sign in. `whoami /groups` confirmed membership of `SG_Finance_Read` only, the user could browse and open files in `\\WIN-QCCH93AVUST\Finance`, and was correctly denied when attempting to create a file in the share.
4. Logged the work as INC0010003 with the same field discipline as the first ticket.

### What I was aiming for in the tickets

- Short description states the symptom, not the fix and not a plea for help. "Unable to log on to workstation, account locked out" rather than "needs help with account unlock".
- Work notes are internal and hold the technical detail and the root cause. Resolution notes are customer visible and written in plain language.
- Both resolution notes end with something that reduces a repeat ticket. On the lockout, that the account also releases itself after 15 minutes. On the onboarding, that modify access needs a separate request.
- Priority is not typed. ServiceNow derives it from the impact and urgency matrix, and the SLA then attaches off the derived priority.

## Issue Encountered and Fix

**Account lockout threshold was 0.** The default domain policy ships with a lockout threshold of 0, which means accounts never lock out no matter how many failed attempts are made. I had to configure the lockout policy before the scenario was possible at all. Obvious in hindsight, but it is the kind of assumption that is easy to carry into a real environment.

**Task 2 password policy had not applied as documented.** While checking the lockout settings I ran `Get-ADDefaultDomainPasswordPolicy` and found `MinPasswordLength` was 6, not the 10 I had recorded as the outcome of Task 2. The same output also showed `ReversibleEncryptionEnabled` set to True, which stores passwords in a recoverable form and should be off. Both were corrected in the Default Domain Policy and reverified from the cmdlet output. I have left the Task 2 README as written and noted the correction here rather than quietly editing it, because the more useful lesson is that I did not verify the applied value at the time and only caught it two tasks later. Reading configuration back from the system rather than from the tool that set it is the habit that surfaced this.

**ServiceNow reference fields.** Caller and Assigned to are reference fields and only accept records that already exist, so typing a name into them fails with an invalid reference error. I created a matching user record for the caller first, then selected it from the lookup. I also initially landed in the ServiceNow IDE, which is the application development interface, rather than the platform UI where incidents live.

## Talking Points

**Incident versus request.** The onboarding scenario is a service request, not an incident. Nothing was broken, it is routine service delivery. In a production instance it would be a Service Catalog item with its own workflow and approval. I logged it on the incident form because the developer instance catalog has no matching item, and I have kept the classification distinction here rather than pretending the record type was correct.

**Scenarios considered and dropped.** I also considered a password reset scenario and a group membership change scenario. Both were dropped as near duplicates of the two above. A password reset exercises the same ADUC account tab and the same ticket shape as the lockout, and a group membership change is the same provisioning and verification path as the onboarding. Two scenarios done properly seemed more useful than four done shallowly.

## Screenshots

| File | What it shows |
|---|---|
| `01-client-lockout-error.png` | Lockout message at the client logon screen after exceeding the threshold |
| `02-aduc-unlock-account.png` | The Unlock account option on the Account tab in ADUC |
| `03-inc0010001-resolved.png` | Resolved lockout incident with caller, classification, priority and resolution |
| `04-inc0010001-work-notes.png` | Work notes for the lockout incident, including root cause and action taken |
| `05-jvelaryon-whoami-groups.png` | `whoami /groups` on the client confirming SG_Finance_Read membership only |
| `06-jvelaryon-write-denied.png` | Access denied when the new user attempts to create a file in the Finance share |
| `07-inc0010003-resolved.png` | Resolved onboarding ticket with work notes and resolution notes |

## Scope

This is a lab built on a single domain controller and one client. The ServiceNow instance is a free developer instance with demo data, so the caller records and assignment groups are stand ins rather than a real service desk configuration. The two scenarios are the tickets I ran, not a full service desk workload.
