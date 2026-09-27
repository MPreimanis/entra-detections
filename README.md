# entra-detections

KQL queries for Microsoft Log Analytics and Microsoft Sentinel that catch common attacks on Microsoft Entra ID accounts: password spraying, MFA fatigue, device code phishing, consent phishing, and the persistence attackers set up afterwards. Each query starts with a comment saying what it finds, the MITRE ATT&CK technique, the data it needs, how to tune it and what to do when it fires.

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

- A Log Analytics workspace, with a diagnostic setting in Entra ID that sends `SignInLogs`, `NonInteractiveUserSignInLogs` and `AuditLogs` to it. Add `UserRiskEvents` for query 04.
- Entra ID P1 or P2 to export the logs. Risk data (queries 04 and 11) needs P2.
- Query 12 needs the Microsoft 365 connector in Microsoft Sentinel, which fills the `OfficeActivity` table.

## Using them

Paste a query into Logs in the workspace, adjust the time range and thresholds at the top, and run it. In Microsoft Sentinel, create a scheduled analytics rule from it to get alerts; queries 01, 02, 03, 06 and 08 make good first alert rules.

Expect to tune. Your own admins, pipelines and credential rotations will show up in queries 08, 09 and 10, and a busy tenant needs a higher threshold in query 02 than a small one.

## Why these

They follow real incidents. Midnight Blizzard (disclosed January 2024) started with a password spray against a test tenant account without MFA and continued through an OAuth application with elevated access. Uber's 2022 breach used repeated push prompts. Storm-2372 used device code phishing in 2025. Most of the queries look for what an attacker does after getting in, because that persistence (a new MFA method, an app credential, a forwarding rule) is what survives a password reset.

## Licence

MIT
