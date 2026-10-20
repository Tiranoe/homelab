# Risk Register

Risks I've identified while building the lab, how I scored them, what controls they map to, and what I'm doing about each one. Every daily log feeds this register.

**Last reviewed:** 2026-10-19 (after Day 5)

## Method

**Scoring.** I rate each risk as if this lab were a small production environment, so the scores reflect real-world impact, not "it's just a lab."

| Score | Likelihood | Impact |
|---|---|---|
| 1 | Unlikely | Minor: inconvenience, easy to recover |
| 2 | Possible | Moderate: outage, rework, or limited exposure |
| 3 | Likely | Severe: domain compromise, data exposure, or long outage |

**Rating** = Likelihood x Impact: **1–2 Low**, **3–4 Medium**, **6–9 High**

**Treatment options**
- **Mitigate:** reduce likelihood or impact with a control
- **Accept:** consciously live with it, with a reason
- **Avoid:** stop doing the thing that creates the risk
- **Transfer:** move it to someone else (insurance, a vendor)

**Control frameworks**
- **NIST SP 800-171 Rev 2** (the basis for CMMC Level 2), shown as `3.x.x`
- **CIS Controls v8**, shown as `CIS x.x`
- **NIST SP 800-53 Rev 5** and **NIST CSF 2.0** where 800-171 doesn't cover the topic

## Summary

| Rating | Total | Open / In progress | Mitigated / Closed | Accepted |
|---|---|---|---|---|
| High | 11 | 6 | 4 | 1 |
| Medium | 6 | 3 | 0 | 3 |
| Low | 4 | 1 | 1 | 2 |
| **Total** | **21** | **10** | **5** | **6** |

**Top priorities:** R-021 (DC01 unreachable), R-002 (passwords not in a password manager; already caused lost credentials), R-015 (static local admin password), R-018 (no audit logging).

## Register

| ID | Risk | Found | L | I | Rating | Controls | Treatment | Status | Next action |
|---|---|---|---|---|---|---|---|---|---|
| R-001 | Evaluation licenses expire: Windows Server (~2027-04-05) and Windows 11 Enterprise (~2027-01-12) | Day 1, 4 | 3 | 1 | Medium (3) | CSF ID.AM-02 | Accept | Accepted | Calendar reminders; rearm or rebuild from logs |
| R-002 | Passwords (Administrator, DSRM, john.nam.admin, lab users, WS01 localadmin) not kept in a password manager. Realized on Day 5: localadmin password lost. | Day 1–2, 5 | 3 | 3 | **High (9)** | 3.5.10 | Mitigate | Open | Move all lab credentials to a password manager, destroy paper copies |
| R-003 | Lab VMs have outbound internet through the host's NAT, so anything malicious tested later could reach the internet | Day 1–2 | 1 | 3 | Medium (3) | 3.13.1, 3.13.6 | Mitigate | Open | Build an isolated attack segment (pfSense VLAN) before using any offensive tools |
| R-004 | Multi-line admin scripts keep running after a failed line (caused duplicate switches on Day 2) | Day 2 | 3 | 2 | **High (6)** | 3.4.3, 3.4.4 | Mitigate | Mitigated | Run changes one command at a time, use `-ErrorAction Stop`, read before running |
| R-005 | Admin commands run on the wrong machine (host and VMs show the same prompt) | Day 2 | 2 | 3 | **High (6)** | 3.4.3 | Mitigate | Mitigated | Run `hostname` before any change |
| R-006 | Domain/forest functional level is Windows Server 2025, so future DCs must be 2025+ | Day 2 | 1 | 1 | Low (1) | 3.4.1 | Accept | Accepted | None |
| R-007 | Hyper-V checkpoints are the only recovery method for DC01; not real AD backups, and rolling back a DC is unsafe with multiple DCs | Day 2 | 2 | 2 | Medium (4) | 800-53 CP-9, CP-10 | Mitigate | Open | Set up Windows Server Backup system-state backups |
| R-008 | Built-in Administrator account still used: well-known target, no individual accountability | Day 3 | 2 | 3 | **High (6)** | 3.3.2, 3.5.1; CIS 4.7, 5.4 | Mitigate | In progress | Use john.nam.admin only; long unique vaulted password; monitor sign-ins. Day 5: "Allow Administrator account lockout" confirmed enabled |
| R-009 | Domain Admins membership grows over time (privilege creep) | Day 3 | 2 | 3 | **High (6)** | 3.1.5, 3.1.7; CIS 5.4 | Mitigate | Mitigated (ongoing) | Monthly membership review (see [Identity and Access](IDENTITY-AND-ACCESS.md)) |
| R-010 | Lab users created without "must change password at next logon," so the admin knows user passwords | Day 3 | 2 | 2 | Medium (4) | 3.5.9 | Accept | Accepted | Lab convenience only; onboarding procedure enforces it for real accounts |
| R-011 | DNS relies on a single external forwarder (1.1.1.1) | Day 3 | 1 | 2 | Low (2) | 800-53 SC-22 | Accept | Accepted | Root hints are the fallback; optional second forwarder |
| R-012 | Host memory at capacity: 85% in use, committed 17.4 of 19.9 GB with two VMs | Day 4 | 3 | 2 | **High (6)** | CSF PR.IR-04 | Mitigate | Open | Max two VMs at a time; upgrade to 2x16 GB DDR4-3200 SODIMM |
| R-013 | Granting Remote Desktop rights to a standard user as a workaround for Enhanced Session | Day 4 | 1 | 2 | Low (2) | 3.1.5, 3.1.12 | Avoid | Closed | Used Basic Session instead |
| R-014 | No DHCP, so every IP and DNS setting is manual; one wrong DNS entry breaks domain join | Day 4 | 2 | 1 | Low (2) | 3.4.2 | Mitigate | Open | DHCP on DC01 with DNS option 10.10.10.10 |
| R-015 | Static, manually set local admin password on WS01; reused passwords enable lateral movement. Day 5: password forgotten with no way to look it up. | Day 4, 5 | 2 | 3 | **High (6)** | 3.5.10; CIS 5.2 | Mitigate | Open | Reset via domain admin, then deploy Windows LAPS |
| R-016 | No MFA for privileged accounts | Review 10/14 | 2 | 3 | **High (6)** | 3.5.3; CIS 6.5 | Accept | Accepted | Isolated lab with no remote access; revisit with hybrid identity |
| R-017 | Account lockout policy was threshold 0 (unlimited guesses) | Review 10/14 | 2 | 2 | Medium (4) | 3.1.8 | Mitigate | In progress | Day 5: set to 5 / 15 min / 15 min in Default Domain Policy and verified; end-to-end lockout test pending (blocked by R-021) |
| R-018 | No centralized audit logging or alerting; privileged actions aren't reviewed | Review 10/14 | 3 | 2 | **High (6)** | 3.3.1, 3.3.2; CIS 8.2 | Mitigate | Open | Advanced audit policy via GPO; Wazuh after the RAM upgrade |
| R-019 | Public GitHub repo could expose passwords, my home IP or employer details | Day 1 | 2 | 3 | **High (6)** | 3.1.22 | Mitigate | Mitigated | Pre-commit checklist below; no work data in the lab |
| R-020 | Cached domain logons let previous users sign in when no DC is reachable; failed attempts against the cache don't count toward lockout | Day 5 | 2 | 2 | Medium (4) | 3.5.2, 3.1.8 | Accept | Accepted | Needed for offline laptops; review "number of previous logons to cache" via GPO later |
| R-021 | DC01 unreachable from WS01 and the host on LDAP (389), so domain sign-ins fail, and nothing alerted me | Day 5 | 3 | 2 | **High (6)** | 800-53 SI-4; CSF DE.CM | Mitigate | Open | Check DC01 services, local port 389, outbound connectivity and firewall |

## Pre-commit checklist (R-019)

Before every push to GitHub:
- [ ] No passwords, recovery keys or secrets in text or screenshots
- [ ] No home public IP, router details or personal information
- [ ] Nothing from my employer: hostnames, network details, screenshots
- [ ] Only lab addresses (10.10.10.0/24) and lab accounts appear

## Change history

| Date | Change |
|---|---|
| 2026-10-14 | Register created from Day 1–4 logs; R-016, R-017, R-018 added during review |
| 2026-10-19 | Day 5: added R-020, R-021; R-002 raised to 9; R-017 in progress; notes on R-008, R-015 |
