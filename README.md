# Homelab

Personal lab on a Dell Latitude 5440 (Intel i5, 16 GB RAM, Windows 11 Pro, Hyper-V) to practice Windows Server, Active Directory, networking and security. I log every session: what I did, what I learned, risks I noticed, and every issue I hit.

## Current lab

| | |
|---|---|
| **Host** | Dell Latitude 5440, 16 GB RAM, Windows 11 Pro |
| **Hypervisor** | Hyper-V |
| **Lab network** | LabNet (internal switch), 10.10.10.0/24, NAT through the host at 10.10.10.1 |
| **Domain** | lab.internal (NetBIOS: LAB), functional level Windows Server 2025 |

| VM | OS | IP | Role | RAM |
|---|---|---|---|---|
| DC01 | Windows Server 2025 Standard (Desktop Experience) | 10.10.10.10 | Domain controller, DNS, Global Catalog | 4 GB |

**Directory layout**

```
lab.internal
└── LAB
    ├── Users          john.nam, alice.wonderland, bob.roberts
    ├── Admins         john.nam.admin (Domain Admins)
    ├── Groups         GG-IT, GG-HR, GG-Finance
    ├── Workstations
    └── Servers
```

## Built so far

- [x] Windows Server 2025 VM on Hyper-V
- [x] Isolated lab network with NAT
- [x] Domain controller for lab.internal with integrated DNS
- [x] DNS forwarder to 1.1.1.1
- [x] OU structure, users, groups and a separate admin account
- [x] Verified least privilege (standard users can't sign in to the DC)

## Roadmap

- [ ] Windows 11 client joined to the domain
- [ ] Group Policy (password policy, lockout, audit settings)
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

## Repo layout

```
logs/          one file per session
screenshots/   evidence referenced in the logs
```
