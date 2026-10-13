---
id: "TOOLS/SQL_Server/08_Operations/Installation_and_Configuration"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Installation and Configuration

**L4 Advanced** · [↑ Operations and Administration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Install SQL Server by script, with only the features you need and a meaningful instance name, configure memory and other settings deliberately, then keep enforcing that configuration with configuration management so the estate never drifts.

### 1. The big picture
Installation used to be a wizard and a runbook. Today you may run 100+ instances across on-premises VMs, cloud IaaS, containers and managed services. Hand-built instances are inconsistent, insecure in places, and slow to build. Good practice has four parts:

1. **Plan**: meaningful names, correct OS and edition, only the features required, one purpose per server (don't over-consolidate).
2. **Automate**: scripted (not manual, not gold-image) installation.
3. **Configure**: set the settings that defaults get wrong, above all memory.
4. **Maintain**: configuration management that detects and repairs drift.

It also ties in availability planning: the architecture chosen must meet the business's requirement at a sensible cost.

### 2. Core concepts and vocabulary
- **Instance** — one SQL Server engine with its own databases, services and configuration. A server can host a *default* instance (`MSSQLSERVER`) and/or *named* instances.
- **Edition** — Enterprise, Standard, Developer, Express, Web: licensing and feature differences.
- **Feature** — optional components: Database Engine, SSIS, SSAS, SSRS, Full-Text, etc. Each is a separate service with its own memory and attack surface.
- **Unattended / scripted install** — repeatable installation driven by a script and a configuration file (PowerShell, `setup.exe /ConfigurationFile`).
- **Gold image** — a pre-built VM image with SQL Server installed; consistent but inflexible.
- **Configuration drift** — the instance's state diverges from the intended standard after the build.
- **Configuration management** — periodic evaluation and correction of state (Puppet, Ansible, PowerShell DSC).
- **DSC resource / manifest / node** — a resource has a *test* method (is it correct?) and a *set* method (make it correct); a manifest/configuration lists the resources and values for a *node* (target server).
- **Buffer pool** — the memory area caching data and index pages; the main memory consumer governed by `max server memory`.
- **CIS benchmark** — a security hardening baseline many organizations adopt.
- **Footprint** — the amount of surface (features, services, memory) an installation exposes.

### 3. How it works, step by step
**1. Plan**
- *Meaningful names*: the instance and database names should say what they host (`sql-prod-timechewer`, database `TimeChewer`), not `SQL01`/`DB03`.
- *OS and edition*: choose the supported OS; the lowest edition that supplies the required features; use Developer for non-production.
- *Features*: install only the Database Engine unless others are needed; install SSIS/SSAS on their own servers where possible.
- *Consolidation*: avoid piling instances onto one server without capacity planning; failure blast radius and resource contention grow.

**2. Script the install**
1. Keep a configuration file or PowerShell/DSC script under source control.
2. Parameters: instance name, edition/product key, features, service accounts, data/log/tempdb/backup paths, collation, sysadmin group.
3. The script installs, then applies the hardening and performance configuration (see below).
4. Run it the same way for every instance.

**3. Configure**
- *Memory*: set `max server memory` (see formula below).
- *Tempdb*: several equally sized data files, on fast storage.
- *Parallelism*: set `max degree of parallelism` and `cost threshold for parallelism` to sensible values.
- *Instant file initialization*, *backup compression default*, *optimize for ad hoc workloads* where appropriate.
- *Hardening*: disable `xp_cmdshell`, minimize `sysadmin`, remove unused features/protocols, patch.

**4. Keep it in the desired state**
1. Define the configuration in code.
2. Apply at build.
3. Evaluate on a schedule (for example every 30 minutes): run each resource's *test*; run *set* only when drift is found.
4. Alert on persistent failures.

### 4. Worked example
**Memory formula.** By default the instance will grow its buffer pool as large as the server allows. Windows, SSIS/SSAS services, antivirus/monitoring/inventory agents and extended stored procedures also need memory; Microsoft recommends leaving about 25% for the OS. The book's rule:

*Max memory = ((server memory − memory needed by other processes) / 100) × 75*

For a dedicated 64 GB server → 48 GB:

```sql
EXEC sys.sp_configure N'show advanced options', 1;  RECONFIGURE;
EXEC sys.sp_configure N'max server memory (MB)', 49152;  -- 48 GB
RECONFIGURE;
```

If several instances share the server, split that figure across them. Note that the setting is more dangerous when *Lock Pages in Memory* is enabled: the OS can no longer trim SQL Server's working set to relieve pressure.

**A DSC configuration** (idea: declarative, idempotent, dependency-ordered):

```powershell
Configuration InstallSql {
    param ($SqlInstanceName = 'MSSQLSERVER', $Edition)
    Import-DscResource -ModuleName SqlServerDsc

    node localhost {
        SqlSetup 'InstallInstance' {
            InstanceName        = $SqlInstanceName
            Features            = 'SQLENGINE'
            SourcePath          = 'C:\SQL Media'
            SQLSysAdminAccounts = @('Administrator')
        }
        SqlConfiguration 'RemoteAdminOff' {
            InstanceName = $SqlInstanceName
            OptionName   = 'remote access'
            OptionValue  = 0
            DependsOn    = '[SqlSetup]InstallInstance'
        }
        SqlLogin 'AddSqlAdmin' {
            Ensure = 'Present'; Name = 'CONTOSO\DbaTeam'; LoginType = 'WindowsGroup'
            InstanceName = $SqlInstanceName
            DependsOn    = '[SqlSetup]InstallInstance'
        }
    }
}
. C:\DSC\SqlInstallDsc.ps1
InstallSql -Edition 'Standard' -OutputPath C:\DSC\Out   # compiles a MOF file
Start-DscConfiguration -Path C:\DSC\Out -Wait -Verbose
```

The first resource ensures the instance exists (does nothing if present). `DependsOn` prevents later resources running before the instance exists. Run on a schedule (`ApplyAndAutoCorrect`), the configuration turns `xp_cmdshell` back off within the interval if someone enables it. (Remember to include a resource for that setting.)

**Parameterize environment-specific values** (server names, paths, databases) in variables/config tables/secret managers, never hard-coded in scripts.

### 5. Under the hood
- **Why default memory is risky**: `max server memory` defaults to ~2 PB (effectively unlimited). The buffer pool grows until the OS signals pressure; then Windows trims the working set, causing paging of SQL Server memory and instability. Non-buffer-pool allocations (thread stacks, CLR, linked server drivers, backup buffers) are *not* limited by `max server memory`, hence the 25%+ headroom.
- **Services**: each feature installs its own Windows service, with its own process and memory. SSIS and SSAS are "other applications" as far as buffer pool sizing is concerned.
- **Setup flow**: `setup.exe` reads a configuration file (or command-line parameters), lays down the binaries and system databases, registers services; post-install steps (sp_configure, logins, trace flags, jobs) are separate.
- **Idempotence**: DSC resources are designed so repeated execution has no effect when the state is right; that is what makes continuous enforcement safe.
- **Gold images vs. scripts**: images bake in one edition and one workload profile (e.g. Enterprise + OLTP defaults on a data warehouse) and need re-building for every patch; scripts parameterize and are patched at the source.
- **Cloud**: in IaaS you still install and configure; in PaaS (Azure SQL Database/Managed Instance) much of the above (patching, backups, HA) is handled for you and configuration surface is smaller.

### 6. Performance and design implications
- **Memory misconfiguration is the commonest installation-time performance problem**: too high → paging and instability; too low → buffer pool churn and `PAGEIOLATCH` waits.
- **Tempdb layout** affects allocation contention (`PAGELATCH_*` on allocation pages).
- **Features not needed cost memory and attack surface**; install on separate servers where practical.
- **Consolidation** saves licences but couples failure and performance; plan capacity for the combined peak.
- **Availability cost-benefit**: compare the cost of downtime to the cost of the HA/DR solution at each "number of nines". The book's example: with downtime cost modelled at $262,810 for 99%, four nines (total ≈ $103k) beats five nines (≈ $302k) and no redundancy, so four nines is the economic target. Presenting this helps owners drop "zero downtime" requests.
- **Standardization** (scripts + DSC) reduces troubleshooting time because every instance is the same.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `setup.exe /ConfigurationFile=…` | Unattended install | Every build |
| `max server memory (MB)` | Caps the buffer pool | Immediately after install |
| `max degree of parallelism`, `cost threshold for parallelism` | Parallelism policy | Post-install tuning |
| `backup compression default` | Compress backups by default | Almost always |
| `xp_cmdshell`, `Ole Automation Procedures`, `clr enabled` | Risky features | Keep disabled unless required |
| `SELECT * FROM sys.configurations` | View all settings | Audit/drift check |
| PowerShell DSC (`SqlServerDsc`), Ansible, Puppet | Enforce desired state | Continuous compliance |
| `dbatools` (`Test-DbaMaxMemory`, `Set-DbaMaxMemory`) | Estate checks and fixes | Auditing many instances |
| Startup trace flags | Engine behavior toggles | Only with a documented reason |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Scripted install | Manual build "just this once": estate becomes inconsistent |
| Gold image | Mixed workloads/editions; frequent patching |
| Configuration management | Only point-in-time hardening; drift reintroduces vulnerabilities |
| Dedicated server per critical instance | Consolidating until one failover can't be absorbed |
| Four nines with cross-site HA/DR | Five nines when cost exceeds downtime cost |

### 9. Common mistakes
- **Manual installs** → inconsistent builds, audit failures → script them.
- **Default instance names like `SQL01`/`DB03`** → can't identify what's where → name by purpose.
- **Installing every feature** → larger attack surface, memory competition → Database Engine only unless required.
- **Leaving `max server memory` at default** → OS starved, paging, instability → set it with headroom for OS and other services.
- **Putting SSIS/SSAS/agents on the same server without budgeting memory** → competition → allocate separately.
- **No configuration management** → someone enables `xp_cmdshell` days later → DSC/Ansible/Puppet evaluate and fix.
- **Not matching architecture to requirements** → spend too much or too little → cost/benefit analysis with the business.

### 10. 🎯 Interview kit
**30-second answer:** I script installs (PowerShell/DSC) with only the features needed and meaningful instance names, then configure memory, tempdb, parallelism and hardening as code. I keep the state enforced with configuration management so drift is automatically corrected, and I size HA/DR from a cost-versus-downtime analysis.

**Structure a longer answer:** planning (names, edition, features, consolidation) → automation (script vs. gold image) → configuration (memory formula, tempdb, MAXDOP) → configuration management (resources: test/set, manifests) → availability requirements → cloud differences.

**Likely follow-ups:**
- *How do you set max server memory?* → Total minus what OS and other processes need, ~75% of the remainder for a dedicated server; split across instances.
- *Why not gold images?* → Inflexible: single edition/profile, many images to patch.
- *What is drift and how do you stop it?* → State diverges after build; DSC/Ansible/Puppet re-evaluate and correct.
- *What are DSC test/set methods?* → Test: is it in desired state; Set: change it if not.
- *Why is `xp_cmdshell` a concern?* → Executes OS commands from SQL; a security hole.

**Signals interviewers look for:** automation mindset, awareness of defaults that hurt, security baseline, drift management, and cost-aware availability design.

### 11. Check yourself
- [ ] Calculate `max server memory` for a dedicated 128 GB server with SSIS on the same box (justify your numbers).
- [ ] Explain gold image vs. scripted installation trade-offs.
- [ ] Describe test and set in a configuration-management resource and why it must be idempotent.
- [ ] List four settings you'd verify on any new instance.
- [ ] Explain how you would justify four nines instead of five to a business owner.
<!-- explainer:end -->

## References

- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — SQL Server Integration Services Design Patterns › Contents
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Parameterization and Environment Configuration
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SQL Server installation › 8.8 #40 Not scripting SQL Server installation
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SQL Server installation › 8 SQL Server installation
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.3 #56 Failing to leave enough memory for other applications
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SQL Server installation › 8.9 #41 Thinking configuration management doesn't apply to SQL Server
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.2 #90 Failing to architect for the requirements

