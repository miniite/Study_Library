---
id: "TOOLS/Hadoop/01_HDFS/HDFS_Operations"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# HDFS Command Line and Administration

**L4 Advanced** · [↑ HDFS](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Day-to-day HDFS operation is a small set of commands and routines — the shell, `fsck`, safe mode, include/exclude files, the balancer, snapshots/`distcp` backups and upgrades — that keep blocks present, replicated and balanced.

### 1. The big picture
HDFS is designed to carry on when machines fail, but it still needs operators: new nodes must be added, old ones retired, health checked, data backed up and software upgraded. This topic is the practical layer on top of the architecture: how to interact with the filesystem from the command line, how to inspect its health, and the routines that a Hadoop administrator runs. It builds on HDFS architecture (what the namenode and datanodes do) and on the HA/federation topic (how the master is protected).

### 2. Core concepts and vocabulary
- **`hdfs dfs` / `hadoop fs`** — the filesystem shell (similar to POSIX commands).
- **`hdfs dfsadmin`** — administrative commands (safe mode, refresh, reports).
- **Safe mode** — a read-only namenode state on startup while block reports arrive.
- **fsck** — filesystem checker; reads metadata from the namenode only.
- **Under-/over-/mis-replicated block** — fewer, more, or wrongly-placed replicas than the policy requires.
- **Corrupt block** — all replicas are corrupt; **missing** — no replicas anywhere.
- **Include / exclude file** — which datanodes/node managers may join, and which are being retired.
- **Snapshot** — read-only point-in-time copy of a subtree (metadata + block list only).
- **`distcp`** — parallel (MapReduce) copy between clusters/filesystems.
- **Pseudo-distributed mode** — all daemons on one machine, `dfs.replication=1`.

### 3. How it works, step by step
**First-time setup (single machine or cluster)**
1. Configure `fs.defaultFS` (namenode URI, port 8020 default) and `dfs.replication` (1 on a single datanode, else 3).
2. Passwordless SSH to the hosts (the control scripts start daemons over SSH; pseudo-distributed is just fully distributed on localhost).
3. Format once: `hdfs namenode -format` (creates empty storage dirs; datanodes aren't formatted, they create their dirs at start).
4. Start: `start-dfs.sh`, `start-yarn.sh`, history server. Check `jps` and web UIs (namenode 50070 in Hadoop 2).

**Startup and safe mode**
1. Namenode loads the latest fsimage and replays the edit log; writes a new checkpoint.
2. It enters safe mode: read-only, collecting block reports.
3. Once the configured fraction of blocks has the minimum replicas (`dfs.namenode.safemode.threshold-pct`, plus an extension time), it leaves safe mode automatically.

**Commissioning nodes:** add to the include file → `hdfs dfsadmin -refreshNodes` and `yarn rmadmin -refreshNodes` → update the `slaves` file (used only by the control scripts) → start daemons → verify in the UI → run the balancer (HDFS does *not* move existing blocks to new nodes by itself).

**Decommissioning:** put in the exclude file → refresh → wait until *Decommissioned* (blocks copied away) → remove from include file → refresh → shut down.

**Upgrade (layout change):** finish any previous upgrade → run a full fsck and keep its output → stop YARN/MR → stop HDFS and back up the namenode directories → install new version → start HDFS with `-upgrade` → sanity check → start YARN → **roll back** or **finalize**. The upgrade uses hard links on datanodes, so it does not double storage; after finalizing there is no rollback.

### 4. Worked example
```bash
hdfs dfs -mkdir -p /user/alice
hdfs dfs -put sales.csv /user/alice/
hdfs dfs -ls -h /user/alice
hdfs dfs -setrep 2 /user/alice/sales.csv

hdfs fsck / | tail -n 20
# Status: HEALTHY
#  Under-replicated blocks: 0   Corrupt blocks: 0   Number of racks: 1

hdfs dfsadmin -safemode get          # Safe mode is OFF
hdfs dfsadmin -report                # capacity, live/dead datanodes

hdfs dfsadmin -allowSnapshot /user/alice
hdfs dfs -createSnapshot /user/alice nightly
hadoop distcp -update hdfs://prod/user/alice hdfs://backup/user/alice
```
Reading the `fsck` summary: *Minimally replicated* should be 100%; *under-replicated* blocks heal automatically (`hdfs dfsadmin -metasave` lists what is queued); *mis-replicated* means the rack policy is violated (e.g., all three replicas on one rack) and is fixed automatically; *corrupt/missing* are the real alarms — data loss.

### 5. Under the hood
- **fsck is metadata-only.** It walks the namespace and the namenode's block map; it does not read blocks or contact datanodes. For missing/corrupt files it can `-move` them to `/lost+found` or `-delete` them; the default is to leave them.
- **Edit log and fsimage on restart:** every transaction is flushed and synced to *all* configured directories before the client receives success. The fsimage is only a periodic checkpoint, because writing a multi-gigabyte image per transaction would be too slow.
- **Safe-mode threshold:** setting `dfs.namenode.safemode.threshold-pct` above 1 makes the namenode stay in safe mode indefinitely — useful to hold a cluster read-only for maintenance.
- **Snapshots:** don't copy data; they record each file's metadata and block list. They protect against accidental deletes but are *not* backups — the blocks live in the same cluster.
- **Replication is not backup.** Bugs, controller faults and human error can destroy all replicas; treat HDFS like RAID. Back up high-priority non-regenerable data with `distcp` to another cluster (ideally a different software version) or another store (e.g., S3).
- **Compatibility rules:** APIs, data/metadata formats and wire protocols each have rules; clients must share the *major* version with servers for wire compatibility, which is what makes rolling upgrades possible in Hadoop 2.
- **Security:** early HDFS had authorization (file permissions) but no authentication; Kerberos was adopted so Hadoop delegates "who are you?" to Kerberos and keeps "what may you do?" itself.

### 6. Performance and design implications
- Run `fsck` and the balancer on a schedule (daily is a sensible default) rather than reactively.
- Balancer is throttled to 1 MB/s per datanode by default; raise only in quiet periods.
- Use space quotas and a snapshot/backup policy for user directories and tell users what it is.
- Plan upgrades on a test cluster first; layout changes can make the namenode refuse to start with "old layout version" until started with `-upgrade`.
- Clean temp files (local and in the MapReduce system directory) before upgrades.
- `-ignoreCrc` allows inspecting a damaged file but should not be used for normal reads.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hdfs dfs -ls/-put/-get/-cat/-rm/-du/-setrep` | Basic file operations | Everyday |
| `hdfs fsck / [-files -blocks -locations]` | Health report | Daily, before/after upgrades |
| `hdfs dfsadmin -report` | Capacity and datanode list | Monitoring |
| `hdfs dfsadmin -safemode get/enter/leave/wait` | Control safe mode | Maintenance, scripts |
| `hdfs dfsadmin -refreshNodes` | Re-read include/exclude | Commission/decommission |
| `hdfs dfsadmin -saveNamespace` | Checkpoint (safe mode) | Before shutdown |
| `hdfs dfsadmin -metasave file` | Dump block replication queues | Debug under-replication |
| `start-balancer.sh -threshold n` | Rebalance | After growth |
| `hadoop distcp` | Parallel copy | Backups, migration |
| `hdfs dfs -createSnapshot` | Point-in-time copy | Protect from accidental deletes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Snapshots for fast undo of user mistakes | Using them as the only backup |
| `distcp` to a second cluster for DR | Backing up everything (prioritise non-regenerable data) |
| Rolling upgrades (Hadoop 2 + HA) | Layout-changing upgrades without a test run |
| Keeping safe mode on for maintenance | Forgetting to leave it (writes fail) |
| Finalizing an upgrade to reclaim space | Finalizing before validation (no rollback) |

### 9. Common mistakes
- Treating three replicas as a backup → bug/human error wipes all → snapshots + `distcp` to another cluster.
- `hdfs namenode -format` on an existing cluster → destroys the namespace → format only once, and protect with config/permissions.
- Terminating many datanodes → missing blocks → decommission via exclude.
- Forgetting the balancer after adding nodes → new nodes idle, old ones full → run it.
- Using `hadoop fs -rmr /` style commands as a superuser → mass deletion → quotas, trash, snapshots.
- Skipping fsck before an upgrade → can't tell pre-existing damage from upgrade damage → save the output and compare.

### 10. 🎯 Interview kit
**30-second answer:** Operations = the shell for data, `fsck`/`dfsadmin` for health, safe mode on restarts, include/exclude + refresh to add or retire nodes, the balancer to even out blocks, snapshots and distcp for protection, and a staged upgrade with finalize/rollback.

**Structure a longer answer:** clarify (routine, incident or upgrade?) → mechanism (what the namenode checks, what each command touches) → trade-offs (snapshot vs backup, speed vs interference) → scenario (retire 10 nodes safely).

**Likely follow-ups:**
- *What does fsck check? Does it read data?* → block presence/replication from namenode metadata; no.
- *What is safe mode?* → read-only startup state until enough blocks are reported.
- *How do you decommission a node?* → exclude file, refreshNodes, wait, remove.
- *Replication vs backup?* → replication handles hardware faults only.
- *How can you roll back an upgrade?* → previous state kept via hard links until finalized.

**Signals interviewers look for:** realistic runbook thinking — verification steps, test cluster, not confusing snapshots or replication with backups.

### 11. Check yourself
- [ ] Format, start and verify a pseudo-distributed HDFS.
- [ ] Interpret every line of an `fsck` summary.
- [ ] Walk through decommissioning including the include/exclude precedence.
- [ ] List the nine upgrade steps and where rollback is still possible.
- [ ] Explain why snapshots are cheap and why they are not backups.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Filesystem check (fsck)
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Entering and leaving safe mode
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › The Command-Line Interface
- *Hadoop: The Definitive Guide* — Tom White — Summary › Formatting the HDFS filesystem
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Security
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › ChecksumFileSystem
- *Hadoop: The Definitive Guide* — Tom White — Summary › Configuring SSH

