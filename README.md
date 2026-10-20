# Homelab

Personal lab on a Dell Latitude 5440 (Intel i5-1345U, 16 GB RAM, Windows 11 Pro, Hyper-V) to practice Windows Server, Active Directory, networking and security. I log every session: what I did, what I learned, risks I noticed, and every issue I hit.

## Current lab

| | |
|---|---|
| **Host** | Dell Latitude 5440, Intel i5-1345U, 16 GB DDR4-3200 (2 of 2 slots used), Windows 11 Pro |
| **Hypervisor** | Hyper-V |
| **Lab network** | LabNet (internal switch), 10.10.10.0/24, NAT through the host at 10.10.10.1 |
| **Domain** | lab.internal (NetBIOS: LAB), functional level Windows Server 2025 |

| VM | OS | IP | Role | RAM |
|---|---|---|---|---|
| DC01 | Windows Server 2025 Standard (Desktop Experience) | 10.10.10.10 | Domain controller, DNS, Global Catalog | 4 GB |
| WS01 | Windows 11 Enterprise (Evaluation) | 10.10.10.20 | Domain-joined workstation | 2–4 GB (dynamic) |

**Directory layout**

```
lab.internal
└── LAB
    ├── Users          john.nam, alice.wonderland, bob.roberts
    ├── Admins         john.nam.admin (Domain Admins)
    ├── Groups         GG-IT, GG-HR, GG-Finance
    ├── Workstations   WS01
    └── Servers
```

## Security and risk

- **[Risk Register](RISK-REGISTER.md):** 21 risks scored by likelihood and impact, mapped to NIST SP 800-171 (CMMC L2), CIS Controls v8 and NIST CSF, each with a treatment decision and status
- **[Identity and Access](IDENTITY-AND-ACCESS.md):** account and group inventory, privileged access, tested sign-in rights, joiner/mover/leaver procedure and access review log

## Known issues

- **DC01 not reachable on LDAP from WS01 or the host** (since 2026-10-19). Troubleshooting in [Day 5](logs/2026-10-19-day-05-account-lockout-policy.md), tracked as R-021.

## Built so far

- [x] Windows Server 2025 VM on Hyper-V
- [x] Isolated lab network with NAT
- [x] Domain controller for lab.internal with integrated DNS
- [x] DNS forwarder to 1.1.1.1
- [x] OU structure, users, groups and a separate admin account
- [x] Verified least privilege (standard users can't sign in to the DC)
- [x] Windows 11 client joined to the domain, regular user sign-in verified
- [x] Risk register and identity and access documentation
- [x] Account lockout policy: 5 attempts / 15 minutes (end-to-end test pending)

## Roadmap

- [ ] Group Policy (password policy, lockout, audit settings)
- [ ] DHCP on DC01
- [ ] Windows LAPS for local admin passwords
- [ ] Disabled Users OU and leaver process
- [ ] Windows Server Backup for DC01 system state
- [ ] RAM upgrade to 32 GB
- [ ] File shares with group-based permissions
- [ ] pfSense with VLANs (after RAM upgrade)
- [ ] Wazuh for security monitoring (after RAM upgrade)
- [ ] Map security settings to NIST 800-171 controls

## Daily logs

| Day | Date | Topic |
|-----|------|-------|
| 1 | 2026-10-07 | [Windows Server 2025 VM on Hyper-V](logs/2026-10-07-day-01-windows-server-vm.md) |
| 2 | 2026-10-12 | [Lab network and first domain controller](logs/2026-10-12-day-02-lab-network-and-domain-controller.md) |
| 3 | 2026-10-13 | [DNS forwarder, OUs, users and groups](logs/2026-10-13-day-03-ous-users-and-groups.md) |
| 4 | 2026-10-14 | [Windows 11 client joined to the domain](logs/2026-10-14-day-04-windows-11-client-domain-join.md) |
| 5 | 2026-10-19 | [Account lockout policy (and a DC that stopped answering)](logs/2026-10-19-day-05-account-lockout-policy.md) |

## Repo layout

```
RISK-REGISTER.md         risks, scoring, control mapping, treatment
IDENTITY-AND-ACCESS.md   accounts, groups, privileged access, access reviews
logs/                    one file per session
screenshots/             evidence referenced in the logs
```
