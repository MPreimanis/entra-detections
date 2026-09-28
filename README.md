# entra-detections

KQL queries for Log Analytics and Microsoft Sentinel that look for common attacks on Entra ID accounts, like password spraying, MFA fatigue, device code phishing and consent phishing, and for what attackers tend to set up once they're in. The comment at the top of each query says what it finds, which MITRE ATT&CK technique it maps to, what data it needs, what to tune and what to do when it fires.

## Queries

| Query | Finds | MITRE ATT&CK |
|---|---|---|
| [01 Break-glass sign-in](queries/01-break-glass-sign-in.kql) | Any use of an emergency access account | T1078.004 |
| [02 Password spray](queries/02-password-spray.kql) | One IP failing across many accounts, and any account that then succeeded | T1110.003 |
| [03 MFA fatigue](queries/03-mfa-fatigue.kql) | Many denied or ignored MFA prompts for one user | T1621 |
| [04 User reported suspicious activity](queries/04-user-reported-suspicious-activity.kql) | MFA prompts the user reported as unexpected | T1621 |
| [05 Legacy authentication](queries/05-legacy-authentication.kql) | Successful sign-ins over protocols that can't do MFA | T1078.004 |
| [06 Device code sign-ins](queries/06-device-code-sign-ins.kql) | Sign-ins through the device code flow | T1528 |
| [07 Consent to application](queries/07-consent-to-application.kql) | User and admin consent grants, with permissions | T1528 |
| [08 App credential added](queries/08-app-credential-added.kql) | New secrets or certificates on apps and service principals | T1098.001 |
| [09 Role assigned outside PIM](queries/09-role-assigned-outside-pim.kql) | Directory roles assigned directly instead of through PIM | T1098.003 |
| [10 Conditional Access changes](queries/10-conditional-access-changes.kql) | Policies created, changed or deleted | T1556.009 |
| [11 MFA method after risky sign-in](queries/11-mfa-method-after-risky-sign-in.kql) | A new MFA method within two hours of a risky sign-in | T1556.006 |
| [12 Inbox forwarding rules](queries/12-inbox-forwarding-rules.kql) | Mailbox rules that forward or redirect mail | T1114.003 |

## What you need

- A Log Analytics workspace, with an Entra ID diagnostic setting sending `SignInLogs`, `NonInteractiveUserSignInLogs` and `AuditLogs` to it. Query 04 also needs `UserRiskEvents`.
- Entra ID P1 or P2 to export the logs. The risk data in queries 04 and 11 needs P2.
- Query 12 reads the `OfficeActivity` table, which comes from the Microsoft 365 connector in Sentinel.

## Using them

Paste a query into Logs in the workspace, adjust the time range and thresholds at the top and run it. In Sentinel you can turn a query into a scheduled analytics rule to get alerts. 01, 02, 03, 06 and 08 are good ones to start with.

You'll need to tune them. Your own admins, pipelines and credential rotations will show up in 08, 09 and 10, and a large tenant needs a higher threshold in 02 than a small one.

## Background

The queries are based on real incidents. Midnight Blizzard (disclosed in January 2024) began with a password spray against a test tenant account that had no MFA, and then used an OAuth application with elevated access. The 2022 Uber breach involved repeated push prompts, and Storm-2372 used device code phishing in 2025. Several queries look at what happens after the first sign-in, such as a new MFA method, an app credential or a forwarding rule, because those still work after the password is reset.

## Licence

MIT
