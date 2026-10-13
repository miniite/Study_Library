---
id: "02_Data_Engineering/02_Storage_and_Formats/File_Block_and_Object_Storage"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# File Block and Object Storage

**L4 Advanced** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** File, block and object storage are three abstractions over the same disks — files give you a directory tree with random reads and writes, blocks give you raw fixed-size chunks for databases, and objects give you immutable key-value blobs that scale almost without limit, which is why data lakes sit on them.

### 1. The big picture

Every storage system you use as a data engineer is built from the same raw ingredients (magnetic disks, SSDs, RAM, networking, CPU). What differs is the *abstraction* presented on top. Choosing the wrong one is a classic source of pain: running a high-rate transactional database on an object store, or leaving a data lake on expensive attached block volumes.

This lesson sits under *Storage and Formats*. It explains the three abstractions, the way each one finds your data, and the economics (storage classes) that matter once volumes get large. The sibling lessons cover the hardware (*Storage Media and Hardware*) and the file formats you place *inside* files or objects (*Parquet Format*).

The one-sentence mental model: **a file system is a tree you navigate, a block device is a numbered array of 4 KiB slots, and an object store is a giant hash map from key to immutable bytes.**

### 2. Core concepts and vocabulary

- **File** — a finite-length stream of bytes that you can append to and read or update at any position (random access).
- **Directory tree** — files are organised as a hierarchy; each directory is stored as metadata (names, permissions, pointers) that the OS follows level by level.
- **Block** — the smallest unit a disk can address and read or write. Historically 512 bytes of usable data, now typically 4,096 bytes (4 KiB), plus bits for error detection/correction.
- **Page** — the unit a database works with; usually a multiple of filesystem blocks (4–16 KiB). To follow one pointer you must fetch the whole block, so on-disk structures are designed to minimise block fetches.
- **SAN (storage area network)** — virtualised block devices served over a network from a storage pool; common on-premises.
- **Cloud block storage (e.g., Amazon EBS)** — SAN-like volumes without the cluster management; the default disk for VMs.
- **IOPS** — I/O operations per second; the other headline number for a volume besides throughput (MB/s).
- **Object** — an immutable file-like blob of any type (CSV, JSON, Parquet, video). Supports finite length, but **not** in-place update or append.
- **Bucket / key** — the top-level container (globally unique name in S3/GCS) and the unique string that identifies an object inside it.
- **Range request** — reading a byte range of an object; the object-store equivalent of a random read.
- **Storage class / tier** — a price/availability/retrieval-cost trade-off for the same object (hot, infrequent access, archive).
- **Hybrid object storage** — an object store that exposes some data-locality or push-down capability (e.g., colocating table data, S3 Select-style filtering).

### 3. How it works, step by step

**Finding a file** (file storage)

1. You give the OS a path such as `/Users/me/output.txt`.
2. The OS starts at root `/`, reads the directory's metadata, finds the entry `Users`, follows its pointer.
3. It repeats for each component until the final pointer leads to the file's data on disk.
4. You can now read or write anywhere in the file; appends extend it up to the limits of the host storage system.

**Finding a block** (block storage)

1. The device exposes a flat array of numbered blocks. Two blocks on the same magnetic track can be read without moving the head; blocks on different tracks need a seek. On SSDs, the "seek" is negligible.
2. A filesystem or database engine sits on top and decides what lives where, managing its own pointers (offsets) and caching them in memory until flushed.
3. Cloud volumes (EBS) place data separately from the host server but in the same availability zone, replicate it to at least two machines, and can take point-in-time snapshots without pausing the disk (later snapshots store only changed blocks).

**Finding an object** (object storage)

1. You call the store with `bucket + key`, e.g. bucket `oreilly-data-engineering-book`, key `project-data/11/23/2021/data.txt`.
2. The store treats the key as an opaque string; there is **no directory traversal**. The slashes only *look* like folders.
3. Writes are a single streamed upload; afterwards the object is immutable. To change it you write a whole new object under the same key.
4. Reads can fetch the whole object or a byte range; many readers in parallel scale throughput.

| Property | File storage | Block storage | Object storage |
|---|---|---|---|
| Addressing | Path in a directory tree | Block number | Bucket + key |
| Update in place | Yes | Yes (at block level) | No — rewrite whole object |
| Append | Yes | Via filesystem | No |
| Typical user | OS, apps, local processing | Databases, VMs | Data lakes, backups, ML data |
| Scaling | One host / cluster | One volume (e.g., up to tens of TiB) | Effectively unbounded, parallel |
| Strength | Familiar semantics | Low latency, high IOPS | Cheap, durable, massively parallel |

### 4. Worked example

Listing and tiering in S3 shows the key-vs-directory idea and lifecycle automation:

```bash
# These "folders" are just key prefixes - the store filters keys by prefix
aws s3 ls s3://acme-lake/events/2026/10/13/

# Copy a file in, then "change" it: there is no in-place edit, only overwrite
aws s3 cp daily.csv s3://acme-lake/raw/daily.csv
aws s3 cp daily_fixed.csv s3://acme-lake/raw/daily.csv   # whole object replaced

# Short-lived access without credentials: signed URL valid for one hour
aws s3 presign s3://acme-lake/raw/daily.csv --expires-in 3600
```

A lifecycle rule that automates tiering (JSON given to `aws s3api put-bucket-lifecycle-configuration`):

```json
{
  "Rules": [{
    "ID": "tier-raw",
    "Filter": {"Prefix": "raw/"},
    "Status": "Enabled",
    "Transitions": [
      {"Days": 30,  "StorageClass": "STANDARD_IA"},
      {"Days": 180, "StorageClass": "GLACIER"}
    ]
  }]
}
```

What happens: objects under `raw/` stay in the standard class for 30 days (hot), move to Infrequent Access (cheaper storage, extra retrieval fee), and after 180 days move to an archive class. Nothing in your pipeline changes — the keys stay the same — only the cost and retrieval behaviour do. The `ls` on a prefix with millions of keys can be slow because the store must scan keys for that prefix, even if the "folder" holds few objects.

### 5. Under the hood

**Why object stores scale.** Because objects are immutable and there are no locks or change-synchronisation requirements, the store can spread a single object's bytes across many disks and serve writes and reads as parallel streams, hiding the disks from you. Write speed scales with the number of concurrent streams (up to vendor quotas); read bandwidth scales with parallel requests, VM count and CPU cores.

**Why they are poor at small updates.** Every change means rewriting an entire object. A workload of many small transactional updates per second is a poor fit — use a transactional database or block storage. A low rate of updates, each touching a large amount of data, works well. Table formats such as Delta Lake and Apache Hudi add update/delete management on top of immutable objects; this is the core idea of the data lakehouse and became necessary once privacy rules (GDPR, CCPA) demanded deletion.

**Block-level design pressure.** Because a disk reads whole blocks, on-disk structures try to maximise *locality* and reduce out-of-page pointers. High fan-out and low height (as in B-Trees) minimise block fetches. Log-structured engines may store records not aligned to block boundaries and address them by absolute offset, so a record can straddle two pages and force two reads.

**Object store–backed filesystems.** Tools such as s3fs and Amazon S3 File Gateway mount a bucket like a disk. File Gateway is clever (it merges portions of objects into a new object server-side), but high-rate transactional writes will still overwhelm the update semantics of the store. Use it for infrequently updated files.

**Hybrid object storage.** Some systems add data-locality or filter push-down: Google's Colossus allows fine-grained block placement that BigQuery uses to colocate a customer's table; S3 Select filters data inside the storage cluster before it crosses the network. The lesson is that low-level control of data placement is still valuable.

**Memory-backed systems.** RAM is volatile; systems that keep hot data there (caches, Apache Druid's SSD-resident segments) need a durable layer. Druid keeps one copy on cluster SSDs (cutting live-storage cost by a factor of three versus triple replication) and uses object storage as its durability layer to recover nodes.

### 6. Performance and design implications

- **Use object storage for the lake**, with large batch reads and writes. Keep objects reasonably large; tiny objects multiply request counts and listing costs.
- **Avoid deep "directory" operations**: listing a prefix is a key filter, not a tree walk, and renames are copy + delete.
- **Pick block storage for databases** where durability and low latency matter. Read the volume's IOPS and throughput limits (some classes reach 64 TiB, 256,000 IOPS, 4,000 MiB/s) and size accordingly.
- **Replication scope matters.** EBS replicates within a zone, so it does not survive an availability-zone failure. One Zone-IA drops availability from 99.9% to 99.5% and loses data if the zone is destroyed.
- **Archive tiers are cheap to enter and expensive to leave.** Deep Archive-style classes can start around \$1 per TB per month but take hours to restore and are designed for data accessed once or twice a year. Retrieval cost in time and money must be part of the design.
- **Use signed URLs** for short-lived sharing and object storage as the default file-exchange mechanism between teams and organisations.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `aws s3 ls s3://bucket/prefix/` | Lists keys that match the prefix | Inspecting "folders"; beware cost on huge buckets |
| `aws s3 cp` / `sync` | Uploads or downloads whole objects | Moving files in/out of the lake |
| `aws s3 presign` | Creates a time-limited signed URL | Temporary access without sharing credentials |
| Lifecycle configuration | Moves or expires objects by age/prefix | Automating hot → IA → archive tiering |
| EBS snapshot | Point-in-time, differential volume backup (stored in S3) | Backing up block volumes without downtime |
| Storage class (`STANDARD`, `STANDARD_IA`, `ONEZONE_IA`, `GLACIER`) | Sets price vs retrieval/availability | Matching cost to access frequency |
| s3fs / File Gateway | Mounts a bucket as a filesystem | Infrequently updated files needing file semantics |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Object storage**: lake, backups, ML corpora, big batch reads/writes, parallel scans | Many small transactional updates per second; you need in-place edits or file locks |
| **Block storage**: database volumes, VM disks, need low latency and high IOPS | Cheap bulk retention of petabytes; cross-zone resilience without extra replication |
| **File storage**: one-time ingestion, exploration, local processing | Building pipeline state on a server's attached disk (use ephemeral environments, put intermediates in object storage) |
| **Infrequent Access / One Zone IA** for data read about monthly | Data you will read often; data you cannot afford to lose with a zone |
| **Archive tiers** for compliance data touched once or twice a year | Anything you might need quickly or often — retrieval fees and delays bite |

### 9. Common mistakes

- Treating S3 prefixes as real directories → slow listings and surprising "rename" costs → design key layouts for prefix filtering and avoid directory-style operations.
- Putting a write-heavy transactional workload on a mounted bucket → throttling and huge object rewrites → use a database on block storage and land snapshots in the lake.
- Moving data to archive tiers without modelling retrieval → big bills and day-long restores when someone needs it → set lifecycle rules by actual access patterns and test a restore.
- Assuming block volumes are zone-proof → data unavailable in an AZ outage → add cross-zone replicas or snapshot copies.
- Keeping pipeline state on a local disk → lost on instance replacement → keep intermediates in object storage.

### 10. 🎯 Interview kit

**30-second answer:** File storage is a directory tree with random read/write; block storage is raw fixed-size blocks used by databases and VMs for low latency; object storage is a key-value store of immutable objects with massive parallel throughput and low cost, which is why data lakes use it. Objects can't be updated in place — you rewrite them — so frequent small updates belong in a database.

**Structure a longer answer:** clarify the workload (read/write mix, update rate, latency, durability) → explain each abstraction's access model → map them to workloads → discuss tiering and cost → close with a scenario (e.g., OLTP on EBS, nightly exports and lake on S3 with lifecycle rules).

**Likely follow-ups:**
- *Are there really folders in S3?* → No, keys are flat strings; slashes only look like folders, and "folder" operations are prefix filters.
- *Why aren't object stores good for updates?* → Immutable; any change rewrites the full object, so many small updates are inefficient, while a few large updates are fine.
- *How do lakehouse formats update data on an object store?* → They add a metadata/transaction layer over immutable files (new files plus a log of what is current).
- *What's the downside of Glacier-type tiers?* → Hours to restore and high retrieval cost; best for rarely accessed data.

**Signals interviewers look for:** you pick abstractions by workload rather than brand name, you know object stores' immutability and prefix semantics, and you account for retrieval costs and failure domains.

### 11. Check yourself

- [ ] Explain how the OS locates `/a/b/c.txt` versus how S3 locates `s3://bucket/a/b/c.txt`.
- [ ] State two reasons object stores scale better than file systems.
- [ ] Choose a storage abstraction for: an OLTP database, a 50 TB Parquet lake, a 7-year compliance archive — and justify each.
- [ ] Write a lifecycle policy that tiers `raw/` after 30 and 180 days, and name what you give up at each step.
- [ ] Explain why a 10:1 compression ratio effectively multiplies both scan throughput and network bandwidth (see *Storage Media and Hardware*).
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Block Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Storage area network
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › File Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object store--backed filesystems
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object lookup
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object stores for data engineering applications
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Example: Apache Druid
- *Database Internals* — Alex Petrov — B-Tree Basics › On-Disk Structures
- *Database Internals* — Alex Petrov — Log-Structured Storage › Disk Access

