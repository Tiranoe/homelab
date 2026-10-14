# Identity and Access Management

Who has access to what in lab.internal, how access is granted and removed, and the results of access reviews. Related risks are in the [Risk Register](RISK-REGISTER.md).

**Last reviewed:** 2026-10-14

## Account inventory

| Account | Type | Location | Groups / privileges | Purpose | Status | Created |
|---|---|---|---|---|---|---|
| Administrator | Built-in, privileged | lab.internal/Users | Domain Admins, Enterprise Admins, Schema Admins, Administrators | Forest creation and break-glass | Enabled (R-008) | 2026-10-12 |
| john.nam.admin | Named admin | LAB/Admins | Domain Admins | My admin work | Enabled | 2026-10-13 |
| john.nam | Standard user | LAB/Users | GG-IT | My everyday account | Enabled | 2026-10-13 |
| alice.wonderland | Standard user | LAB/Users | GG-HR | Test user (HR) | Enabled | 2026-10-13 |
| bob.roberts | Standard user | LAB/Users | GG-Finance | Test user (Finance) | Enabled | 2026-10-13 |
| krbtgt | Built-in service | lab.internal/Users | n/a | Signs Kerberos tickets | Disabled (by design) | 2026-10-12 |
| Guest | Built-in | lab.internal/Users | Domain Guests | Not used | Disabled (by default) | 2026-10-12 |
| WS01\localadmin | Local admin | WS01 only | WS01\Administrators | Initial setup of WS01 | Enabled (R-015) | 2026-10-14 |

**Not an AD account:** the DSRM password is a separate local recovery login on DC01, used only to boot into Directory Services Restore Mode (R-002).

## Groups

| Group | Scope / type | Location | Members | Purpose |
|---|---|---|---|---|
| GG-IT | Global security | LAB/Groups | john.nam | IT staff role |
| GG-HR | Global security | LAB/Groups | alice.wonderland | HR role |
| GG-Finance | Global security | LAB/Groups | bob.roberts | Finance role |

**Naming convention:** `GG-` = global group, used for roles. Permissions on resources will be granted to groups, never to individual users.

## Privileged access

| Group | Members | Expected | Notes |
|---|---|---|---|
| Domain Admins | Administrator, john.nam.admin | 2 or fewer | Verified 2026-10-14 |
| Enterprise Admins | Administrator | Administrator only | Verified 2026-10-14; forest-wide rights, should stay near-empty |
| Schema Admins | Administrator | Administrator only | Verified 2026-10-14; only needed for schema changes |

![Domain, Enterprise and Schema Admins membership](screenshots/2026-10-14-privileged-groups-review.png)

**Principles in use**
- **Separate admin accounts:** everyday work as john.nam, admin work as john.nam.admin (800-171 3.1.5, 3.1.6; CIS 5.4)
- **Least privilege:** standard users get no admin rights anywhere
- **Built-in Administrator** is for break-glass use only (R-008, in progress)

## Sign-in rights (tested)

| Account | DC01 local sign-in | WS01 local sign-in | WS01 via Remote Desktop |
|---|---|---|---|
| john.nam (standard) | **Blocked** (tested Day 3) | **Allowed** (tested Day 4) | **Blocked** (seen Day 4) |
| john.nam.admin | Allowed (Domain Admin) | Allowed (Domain Admin) | Allowed (Domain Admin) |

- DC01 is protected by **"Allow log on locally"** in the Default Domain Controllers Policy, which is limited to admin-type groups.
- Remote access to WS01 requires **"Allow log on through Remote Desktop Services"** (Remote Desktop Users group), which standard users don't have.

## Joiner / Mover / Leaver procedure

**Joiner (new user)**
1. Create the account in **LAB/Users** using the `first.last` naming format
2. Set a temporary password with **"User must change password at next logon"** checked (3.5.9)
3. Add to the role group for their job (GG-HR, GG-Finance, GG-IT), not to individual permissions
4. If they need admin rights, create a separate `first.last.admin` account in **LAB/Admins** after approval
5. Record the account in the inventory above

**Mover (role change)**
1. Remove the old role group **before** adding the new one, so access doesn't stack up
2. Re-check any admin account they hold

**Leaver (departure)**
1. **Disable** the account immediately (don't delete it yet)
2. Remove it from all groups
3. Reset the password
4. Move it to a **Disabled** OU (planned) and delete after 30 days
5. Disable or remove any matching `.admin` account

**PowerShell reference**
```powershell
# Leaver: disable and strip group membership
Disable-ADAccount -Identity bob.roberts
Get-ADPrincipalGroupMembership bob.roberts | Where-Object Name -ne "Domain Users" | ForEach-Object { Remove-ADGroupMember -Identity $_ -Members bob.roberts -Confirm:$false }
```

## Access reviews

**How I review**
```powershell
# Privileged group membership
"Domain Admins","Enterprise Admins","Schema Admins" | ForEach-Object { "--- $_ ---"; Get-ADGroupMember $_ | Select-Object -ExpandProperty SamAccountName }

# All lab users, last sign-in and password age
Get-ADUser -Filter * -SearchBase "OU=LAB,DC=lab,DC=internal" -Properties LastLogonDate, PasswordLastSet, Enabled | Select-Object SamAccountName, Enabled, LastLogonDate, PasswordLastSet

# Accounts with no sign-in in 30+ days
Search-ADAccount -AccountInactive -TimeSpan 30.00:00:00 -UsersOnly | Select-Object SamAccountName, LastLogonDate
```

**Review log**

| Date | Scope | Findings | Actions |
|---|---|---|---|
| 2026-10-14 | Domain, Enterprise and Schema Admins; all lab accounts | Domain Admins = Administrator, john.nam.admin. Enterprise Admins and Schema Admins = Administrator only. All appropriate. Built-in Administrator still used for admin work. WS01 has a static local admin password. | Opened R-008 and R-015; next review 2026-11-14 |
