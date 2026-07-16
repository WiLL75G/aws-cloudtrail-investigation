# AWS Cloud Security Investigation

Reading CloudTrail as an analyst, not an admin. Eleven events, one unknown username, and two misconfigurations found in the account's own audit trail.

## At a Glance

| Field | Detail |
| --- | --- |
| Investigation Type | Cloud security review, CloudTrail event analysis |
| Platform | AWS, account 446643829023 |
| Region | eu-north-1, Europe Stockholm |
| Trail | soc-investigation-trail, multi region |
| Log Storage | S3, aws-cloudtrail-logs-446643829023-5598fc6c |
| Events Reviewed | 11 management events |
| Outcome | No unauthorised access, 2 misconfigurations documented |

## What Happened

A CloudTrail trail was created across all regions, and every management event in the account was reviewed and given a verdict.

Nothing malicious was found. What was found is the account's own security posture written into its logs: the root account being used without MFA, and log file validation switched off.

The interesting part of a cloud investigation is not the volume. Eleven events is nothing. It is that every event looks like a username doing a thing, and half the time the username is not a person.

Scope stated plainly: this is a personal AWS account with a small event history, not an enterprise estate. What it demonstrates is the reading, not the scale.

## CloudTrail Setup

![AWS Console](./screenshots/01_aws_console_dashboard.png)

Console accessed, CloudTrail opened, multi region trail created and confirmed logging.

Multi region is not optional. A single region trail is a trail with blind spots, and an attacker who knows which region you watch will work in one of the other thirty.

## Trail Verification

![CloudTrail Active](./screenshots/04_cloudtrail_active.png)

Status confirmed as Logging. S3 bucket created for storage. Multi region coverage confirmed.

Verify the trail before trusting the evidence. A disabled trail is not an absence of activity, it is an absence of visibility, and those look identical from the console.

Disabling CloudTrail is itself an attacker move. If a trail is off, the first question is not what happened, it is who turned it off and when.

## Event History Review

![Event History](./screenshots/05_event_history.png)

![All Events](./screenshots/06_all_events.png)

All 11 management events across the last 90 days reviewed.

Two distinct identities appear: root, and a user called onboarding.

Root actions traced to the account owner. The onboarding user needed explaining before anything could be cleared.

Every event gets a verdict. An event nobody looked at is not a clean event, it is an unread one.

## Investigating the Unknown User

![Event Details](./screenshots/07_event_details_p1.png)

The onboarding user performed an AssociateDefaultView event.

A username nobody created is exactly the thing that ends an investigation early in the wrong direction. Reading the full JSON is what settles it.

The record showed:

User type: AssumedRole, an AWS service role rather than a human identity.

Source IP: resource-explorer-2.amazonaws.com, an AWS internal service endpoint.

Verdict: AWS Resource Explorer configuring itself. Not suspicious.

This is the cloud specific skill. On a Linux box a username is a person. In AWS a username is often a service assuming a role, and the only way to tell is the identity type and the source in the JSON. The console view shows you the name. The JSON shows you what the name is.

## Finding, Root Used Without MFA

![CreateTrail Event](./screenshots/08_createtrail_p1.png)

![CreateTrail JSON](./screenshots/08_createtrail_p2.png)

The CreateTrail event was performed by the root user from 197.254.137.4.

The JSON returned `mfaAuthenticated: false`.

Root without MFA is the finding that outranks everything else in an AWS account. Root cannot be restricted by IAM policy, cannot be limited in scope, and cannot be locked out of anything. It is the account. One password between an attacker and total control, with no second factor behind it.

The source IP traces to the account owner, so this is a misconfiguration and not an intrusion. It is still the highest severity item here, because the control that would stop an intrusion is the one that is missing.

## Finding, Log File Validation Disabled

Log file validation was not enabled on the trail.

Validation is what makes CloudTrail logs evidence rather than just records. Without it, a log file can be altered after the fact and nothing detects the change.

An audit trail that cannot prove it has not been edited is an audit trail an attacker can rewrite. This finding is quieter than the root one and it undermines every other finding in the account, because it means none of the evidence can be proven intact.

## Findings Summary

| # | Finding | Severity | Status |
| --- | --- | --- | --- |
| 1 | Root account used without MFA | High | Remediation required |
| 2 | Log file validation not enabled | Medium | Remediation required |
| 3 | onboarding user activity | Cleared | AWS Resource Explorer service, verified in JSON |
| 4 | CreateDefaultVpc with no username | Cleared | AWS automated activity |

Two of these are findings and two are cleared. Both cleared entries were investigated to a verdict, not assumed.

## Observations

| Type | Value | Verdict |
| --- | --- | --- |
| User | root | Used without MFA, high severity |
| Source IP | 197.254.137.4 | Account owner, expected |
| User | onboarding | AWS Resource Explorer service role, cleared |
| Trail | soc-investigation-trail | Created for this investigation |

## MITRE ATT&CK Relevance

| Technique | ID | Why It Applies |
| --- | --- | --- |
| Valid accounts, cloud accounts | T1078.004 | Root used for management tasks is the account an attacker wants |
| Impair defences, disable cloud logs | T1562.008 | Log validation off means log tampering would go undetected |

Mapping note: these are the techniques the misconfigurations expose the account to. Neither was observed. No adversary activity was present in the event history.

## Analyst Conclusion

11 CloudTrail events reviewed, every one assigned a verdict.

No unauthorised access detected.

Root account used without MFA. Highest severity finding in the account.

Log file validation disabled, meaning tamper detection is not active on the audit trail.

The onboarding user is an AWS service role, confirmed from the identity type and source endpoint in the JSON.

All activity attributable to the account owner or to AWS automation.

## Recommended Response

Enable MFA on root immediately. It is the single control with the largest blast radius in the account.

Stop using root for routine work. Create an IAM admin user and reserve root for the handful of tasks that require it.

Enable log file validation on the trail so the evidence can prove it is intact.

Enable GuardDuty so detection is not dependent on someone reading event history by hand.

## What This Lab Demonstrates

Configuring a multi region CloudTrail and verifying it before trusting its output.

Reading CloudTrail JSON rather than the console summary.

Distinguishing an AWS service role from a human identity, which is the mistake cloud investigations turn on.

Investigating an unknown username to a verdict instead of clearing it or escalating it on the name alone.

Identifying the two misconfigurations that matter most in an AWS account and explaining why they rank the way they do.

Assigning a verdict to every event, including the ones that turned out clean.

## Repository Structure

```
aws-cloud-security-investigation-lab/
├── README.md
└── screenshots/
    ├── 01_aws_console_dashboard.png
    ├── 02_cloudtrail_homepage.png
    ├── 03_cloudtrail_create_form.png
    ├── 04_cloudtrail_active.png
    ├── 05_event_history.png
    ├── 06_all_events.png
    ├── 07_event_details_p1.png
    ├── 07_event_details_p2.png
    ├── 07_event_details_p3.png
    ├── 08_createtrail_p1.png
    ├── 08_createtrail_p2.png
    └── 08_createtrail_p3.png
```

---

[![LinkedIn](https://img.shields.io/badge/LinkedIn-WilliamInCyber-blue?style=flat&logo=linkedin)](https://linkedin.com/in/WilliamInCyber)
[![X](https://img.shields.io/badge/X-WilliamInCyber-black?style=flat&logo=x)](https://x.com/WilliamInCyber)
