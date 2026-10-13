---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce/Cluster_Setup_and_Administration"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Cluster Setup and Administration

**L4 Advanced** · [↑ YARN and MapReduce on Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Standing up a Hadoop cluster means choosing a certified OS/Java/Hadoop combination, writing a small set of site files (`core-site`, `hdfs-site`, `yarn-site`, `mapred-site`), sizing memory and CPU for node managers, formatting HDFS once and starting the daemons — then running it with commissioning, monitoring and security routines.

### 1. The big picture
Hadoop has "a bewildering number of configuration properties", but only a short list must be defined — or at least understood — on any real cluster. This lesson is a guided tour of that list: installation prerequisites, the configuration files and the properties in them, the three run modes, the daemon addresses you will meet in logs and firewalls, memory/CPU sizing for YARN, and the first-run sequence. In production most operators use a cluster manager (Cloudera Manager, Apache Ambari) which automates exactly these steps; knowing the underlying settings is what lets you debug it.

It builds on the HDFS lessons (namenode/datanode directories, safe mode) and on YARN (containers, schedulers), and precedes day-two operations: commissioning, upgrades, backups.

### 2. Core concepts and vocabulary
- **Site files** — XML configuration in `etc/hadoop`: `core-site.xml` (common), `hdfs-site.xml`, `yarn-site.xml`, `mapred-site.xml`.
- **Default files** — `core-default.xml`, `hdfs-default.xml`, … document every property and default.
- **Standalone / pseudo-distributed / fully distributed** — single JVM; all daemons on one host; daemons on a cluster.
- **`slaves` file** — hosts the control scripts SSH into; *not* used by daemons.
- **Include/exclude files** — which hosts may connect (`dfs.hosts`, `yarn.resourcemanager.nodes.include-path`).
- **Rack awareness script** — maps hosts to racks for placement/locality.
- **Kerberos** — authentication mechanism used by Hadoop for secure clusters.

### 3. How it works, step by step
**Installation and first start**
1. Pick an OS + Java + Hadoop combination certified by your distribution vendor; install Java and set `JAVA_HOME`.
2. Create a dedicated `hdfs` (and `yarn`) user; unpack Hadoop on every node.
3. Set up passwordless SSH from the admin node (control scripts start daemons over SSH, even in pseudo-distributed mode where the host is localhost).
4. Write the site files (below) and distribute them identically.
5. Format HDFS once, as the `hdfs` user: `hdfs namenode -format`. Datanodes are not formatted — they create storage directories on first start — and capacity is determined by the datanodes you add.
6. Start: `start-dfs.sh`, `start-yarn.sh`, `mr-jobhistory-daemon.sh start historyserver`. (`--config <dir>` or `HADOOP_CONF_DIR` if configuration lives elsewhere.)
7. Verify with `jps`, the web UIs, then create user directories and a user home with proper permissions.

**Run modes** (minimal properties)
| Component | Property | Standalone | Pseudo-distributed | Fully distributed |
|---|---|---|---|---|
| Common | `fs.defaultFS` | `file:///` | `hdfs://localhost/` | `hdfs://namenode/` |
| HDFS | `dfs.replication` | n/a | `1` | `3` |
| MapReduce | `mapreduce.framework.name` | `local` | `yarn` | `yarn` |
| YARN | `yarn.resourcemanager.hostname` | n/a | `localhost` | `resourcemanager` |

### 4. Worked example
Typical site files for a small cluster:
```xml
<!-- core-site.xml -->
<property><name>fs.defaultFS</name><value>hdfs://namenode/</value></property>

<!-- hdfs-site.xml -->
<property><name>dfs.namenode.name.dir</name><value>/disk1/hdfs/name,/remote/hdfs/name</value></property>
<property><name>dfs.datanode.data.dir</name><value>/disk1/hdfs/data,/disk2/hdfs/data</value></property>
<property><name>dfs.namenode.checkpoint.dir</name><value>/disk1/hdfs/namesecondary,/disk2/hdfs/namesecondary</value></property>

<!-- yarn-site.xml -->
<property><name>yarn.resourcemanager.hostname</name><value>resourcemanager</value></property>
<property><name>yarn.nodemanager.local-dirs</name><value>/disk1/nm-local-dir,/disk2/nm-local-dir</value></property>
<property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property>
<property><name>yarn.nodemanager.resource.memory-mb</name><value>16384</value></property>
<property><name>yarn.nodemanager.resource.cpu-vcores</name><value>16</value></property>
```
To see what a running daemon actually uses, open its `/conf` page, e.g. `http://resource-manager-host:8088/conf`: it shows the merged site + default configuration and which file each property came from.

### 5. Under the hood
- **Directories with different semantics:** `dfs.namenode.name.dir` writes a *copy* of the metadata in each directory (redundancy; include an NFS mount); `dfs.datanode.data.dir` *round-robins* blocks across directories (performance; one per disk); `dfs.namenode.checkpoint.dir` holds the secondary's checkpoint copies; `yarn.nodemanager.local-dirs` holds containers' scratch space and map output (spread across disks).
- **Memory sizing for node managers.** Each Hadoop daemon defaults to ~1 GB heap (datanode + node manager = 2 GB). Subtract that and the OS and other processes, and give the rest to `yarn.nodemanager.resource.memory-mb` (default 8192 MB, usually too low). Task containers draw from this pool, so concurrency = pool ÷ container size, not a fixed slot count.
- **CPU sizing.** `yarn.nodemanager.resource.cpu-vcores` = cores minus one per long-running daemon. Tasks default to 1 vcore (`mapreduce.map.cpu.vcores`). CPU is only *accounted* by default; enforce with `LinuxContainerExecutor` + cgroups.
- **Daemon addresses and ports** (defaults): namenode RPC `8020` (from `fs.defaultFS`), namenode web UI `50070`, datanode IPC `50020`, resource manager RPC `8032`, scheduler `8030`, resource-tracker `8031`, admin `8033`, RM web `8088`, job history RPC `10020` and web `19888`. `*.bind-host` properties make a server listen on all interfaces (`0.0.0.0`) while advertising a specific name — needed in multi-homed/cloud setups.
- **Master-node sizing.** The namenode needs RAM proportional to the number of files and blocks (~150 bytes per object), so size its heap for growth; the secondary/standby needs similar memory.
- **Security.** Early Hadoop assumed cooperating users: file permissions guard against accidents, not impersonation. Kerberos provides authentication; Hadoop keeps authorization (who may do what). Other enhancements include encrypted data transfer (`dfs.encrypt.data.transfer`).
- **Other settings:** mount data disks `noatime`; configure the scheduler queues; consider short-circuit local reads; raise `reduce.slowstart` for big jobs.

### 6. Performance and design implications
- Disk layout is the first-order tuning knob: many independent disks per node, JBOD rather than RAID for datanodes (HDFS already replicates).
- Keep memory settings consistent: sum of containers ≤ node manager pool ≤ physical RAM − daemons − OS.
- The `slaves` file only affects scripts; forgetting to update it means new nodes aren't restarted by `start-dfs.sh` even though they work.
- Put the namenode, resource manager and standby on dedicated masters on large clusters; co-locating them with workers saves hardware only on small clusters.
- Use a rack topology script so replica placement and locality work.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hdfs namenode -format` | Initialise namenode storage | Once, new cluster only |
| `start-dfs.sh` / `start-yarn.sh` | Start daemons across hosts via SSH | Cluster start |
| `jps` | List Java daemons | Verify start |
| `/conf` web page | Effective configuration | Debug config drift |
| `yarn.nodemanager.resource.memory-mb` / `cpu-vcores` | Capacity per node | Sizing |
| `yarn.nodemanager.aux-services=mapreduce_shuffle` | Enables the shuffle handler | Required for MapReduce |
| `dfs.hosts` / `.exclude` + `-refreshNodes` | Membership control | Commission/decommission |
| `net.topology.script.file.name` | Rack mapping | Multi-rack |
| `*.bind-host` | Listen on all interfaces | Cloud/multi-homed |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| A cluster manager (Ambari/CM) for production | Hand-editing XML on dozens of nodes |
| Pseudo-distributed for development | Benchmarks or capacity planning |
| JBOD datanode disks | RAID-ing datanode disks (waste and slower) |
| Dedicated master nodes | Tiny clusters where cost dominates |
| Larger node manager pools on big machines | Leaving room for page cache and OS too little |

### 9. Common mistakes
- Formatting an existing namenode → destroys the namespace → format only on a new cluster.
- Leaving `yarn.nodemanager.resource.memory-mb` at 8 GB → wasted RAM → set explicitly.
- Different configs on different nodes → odd, node-specific failures → push identical files and check `/conf`.
- Not enabling the shuffle aux-service → reducers can't fetch map output → set `mapreduce_shuffle`.
- Assuming CPU limits are enforced → noisy neighbours → cgroups.
- Binding services to `127.0.0.1` or default host names in cloud → unreachable from clients → use `bind-host` and proper addresses.

### 10. 🎯 Interview kit
**30-second answer:** Install certified Java/Hadoop, configure `core/hdfs/yarn/mapred-site.xml` (`fs.defaultFS`, namenode/datanode dirs, resource manager host, node-manager memory and vcores), format HDFS once, start HDFS then YARN, and verify via `jps` and the web UIs.

**Structure a longer answer:** clarify (size, HA, security) → config files and key properties → capacity sizing → startup and verification → ongoing admin (commission, balance, monitor).

**Likely follow-ups:**
- *How do you size node-manager memory?* → RAM − daemons − OS, rest to containers.
- *Where do multiple `name.dir` entries matter?* → redundancy, include a remote disk.
- *How do you check the effective config?* → `/conf` page.
- *What's the `slaves` file for?* → control scripts only.
- *How is Hadoop secured?* → Kerberos authentication + HDFS permissions/ACLs + encryption.

**Signals interviewers look for:** you can state the handful of properties that must be set and why, and you separate authentication from authorization.

### 11. Check yourself
- [ ] Write minimal site files for a 3-node cluster.
- [ ] Compute node-manager memory for a 64 GB node running datanode and node manager.
- [ ] Explain redundancy vs round-robin for the storage directory properties.
- [ ] List the daemons and web UIs you expect after startup.
- [ ] Explain the role of the `slaves` file vs include/exclude files.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — N
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Cluster Setup and Installation
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Appendix B. Cloud Networking
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Security and Privacy › Technology

