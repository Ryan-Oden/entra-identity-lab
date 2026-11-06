# Lab 01: Microsoft Entra Cloud Sync

**Date:** Oct 30 to Nov 2, 2026
**SC-300 domain:** Implement and manage user identities (hybrid identity)
**License:** Sync itself is included with Entra ID Free. Password writeback (not covered here) needs Entra ID P1.

## Goal

Sync users from an on-premises Active Directory domain to Microsoft Entra ID with Cloud Sync and password hash sync, then confirm a synced user can sign in to Entra.

## Why Cloud Sync for this lab

- Lightweight agent, with the configuration managed in the Entra portal instead of on a server.
- Microsoft is putting new sync capabilities into Cloud Sync.
- Connect Sync would be the right choice if the scenario needed device objects (hybrid join), pass-through authentication, or complex attribute-based filtering. None of those apply here yet.

## Prerequisites

- A KDS root key in AD (required for the group managed service account the agent runs as).
- A UPN suffix in AD that matches a domain verified in Entra, so synced users get a routable sign-in name.
- A dedicated OU for the users in sync scope.
- An admin with the **Hybrid Identity Administrator** role (least-privileged role for sync setup).

## Steps

1. Created the KDS root key, added the alternate UPN suffix, and created the sync-scope OU.
2. Installed the Cloud Sync provisioning agent on a domain-joined member server (not the domain controller). The installer created a gMSA for the agent.
3. In the Entra admin center, created a Cloud Sync configuration for the domain.
4. Scoped the configuration to the sync OU only.
5. Enabled password hash sync and turned the configuration on.
6. Used on-demand provisioning to sync one test user.

## Verification

- The test user appeared in Entra as a synced (on-premises) user.
- The test user signed in to Entra with the routable UPN and their AD password.

## What broke and what I learned

- **Sign-in failed at first with error 50126 (invalid username or password).** On-demand provisioning created the user right away, but the password hash arrives through the regular sync cycle after the configuration is enabled. A synced user getting 50126 is a signal to check password hash sync.
- Cloud Sync has no "force a sync" cmdlet. Connect Sync does: `Start-ADSyncSyncCycle -PolicyType Delta`.
- The "Directory Synchronization Accounts" role is for the sync service account, not for people. Admins setting up sync should use Hybrid Identity Administrator.
- **Source of authority wins:** attributes on synced users are changed in on-prem AD, not in Entra.

## Okta comparison

- The Cloud Sync agent is close to the Okta AD Agent: lightweight, configured from the cloud, and you can run more than one for high availability.
- **Where it breaks:** the Okta AD Agent can also do delegated authentication (checking passwords against AD in real time). In Entra that's pass-through authentication, a separate agent that comes with Connect Sync, not Cloud Sync.
