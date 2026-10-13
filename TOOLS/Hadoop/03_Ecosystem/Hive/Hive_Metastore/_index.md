---
id: "TOOLS/Hadoop/03_Ecosystem/Hive/Hive_Metastore"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Hive Metastore and Services

**L5 Expert** · [↑ Hive](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The Hive metastore is the shared catalogue that maps table names to schemas and HDFS locations, and HiveServer2 is the front door that lets many remote clients run Hive queries through JDBC/ODBC/Thrift.

### 1. The big picture
A Hive query needs two things: the data (files in HDFS) and a description of it (which columns, which types, which directory, which file format). The description is **metadata**, and it lives in the **metastore**. Because the metastore is just a relational database with a service in front, other engines — Impala, Spark SQL, Presto/Trino, Pig via HCatalog — can read the same catalogue, so a table defined once is visible everywhere. That makes the metastore the de-facto catalogue of the Hadoop ecosystem.

Around it sit the **services** you launch with `hive --service <name>`: the CLI, `hiveserver2`, `beeline`, a standalone `metastore`, and others. This lesson explains how those pieces are wired, and the three metastore deployment modes you choose between.

### 2. Core concepts and vocabulary
- **Metastore service** — the process that exposes metadata operations (create table, get partitions…) over Thrift, default port 9083.
- **Metastore database (backing store)** — the relational DB holding the metadata (Derby, MySQL, PostgreSQL, Oracle).
- **Embedded metastore** — metastore code *and* a Derby database run inside the Hive client process; only one session at a time, local files `metastore_db`.
- **Local metastore** — service in-process with Hive, but the database is a separate RDBMS; many sessions possible.
- **Remote metastore** — the metastore is its own process; Hive clients connect over Thrift. The production norm.
- **HiveServer2** — a server that accepts client connections over Thrift (port 10000 by default, `hive.server2.thrift.port`), with authentication and multi-user concurrency.
- **Beeline** — command-line client that connects to HiveServer2 over JDBC (or runs embedded).
- **HCatalog** — exposes the metastore's table schemas to other tools such as Pig and MapReduce so they can refer to tables by name.
- **Kerberos** — the usual authentication mechanism on a secured Hadoop cluster (ticket-granting ticket → service ticket → service request).

### 3. How it works, step by step
**Deployment modes**

| Mode | Metastore code | Database | Concurrency | Typical use |
|---|---|---|---|---|
| Embedded | Inside Hive JVM | Embedded Derby | Single user | Learning, laptop |
| Local | Inside Hive JVM | Separate RDBMS | Multi-user, but every client needs DB credentials | Small teams |
| Remote | Separate service | Separate RDBMS | Multi-user; clients don't touch the DB | Production |

**A remote-metastore query**
1. A BI tool opens a JDBC connection `jdbc:hive2://host:10000/dbname` to HiveServer2.
2. HiveServer2 authenticates the user (Kerberos/LDAP/none).
3. The compiler asks the metastore service (Thrift, port 9083) for the table schema, partitions, location and SerDe.
4. The metastore reads/writes its backing database and answers.
5. HiveServer2 builds the plan and submits it to YARN; results stream back to the client.

**Clients**
- *Thrift client* — any language with Thrift support.
- *JDBC driver* — type 4 pure-Java driver (`org.apache.hive.jdbc.HiveDriver`), URI `jdbc:hive2://host:port/db`; `jdbc:hive2://` alone means **embedded mode** (Hive runs inside the application JVM).
- *ODBC driver* — for tools such as Excel/Tableau.

### 4. Worked example
```bash
# start a standalone metastore on port 9083
hive --service metastore -p 9083 &

# start HiveServer2 (defaults to port 10000)
hive --service hiveserver2 &

# connect with Beeline
beeline -u jdbc:hive2://localhost:10000/default -n alice
```
```xml
<!-- hive-site.xml on a client pointing to a remote metastore -->
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://metastore-host:9083</value>
</property>
```
```sql
-- see where Hive thinks a table lives and what format it uses
DESCRIBE FORMATTED sales;
```
`DESCRIBE FORMATTED` is answered entirely from the metastore: location, input/output format, SerDe, partition columns, table type (`MANAGED_TABLE` or `EXTERNAL_TABLE`). If `hive.metastore.uris` is unset the client falls back to embedded mode and you will see an empty catalogue and a `metastore_db` folder appear in the current directory — a classic confusion.

### 5. Under the hood
- The metastore database contains normalised tables (`DBS`, `TBLS`, `SDS`, `COLUMNS_V2`, `PARTITIONS`, `PARTITION_KEYS`…). A table with 100,000 partitions means 100,000 rows in `PARTITIONS` plus storage-descriptor rows; fetching them all at planning time can become the bottleneck, which is why partition pruning is done in the metastore query.
- The metastore is also where the **compactor** for transactional tables is driven from, and where table statistics (row counts, column min/max, distinct estimates) are stored for the cost-based optimizer.
- HiveServer2 improves on the original HiveServer by adding authentication and multi-user concurrency; each session has its own configuration.
- **Kerberos** flow: user runs `kinit` → Authentication Server returns a TGT (valid ~10 h, renewable to a week) → TGT is exchanged at the Ticket-Granting Server for a service ticket → the service (NameNode, ResourceManager, HiveServer2) accepts it. Unattended jobs use a **keytab** (`ktutil`) with `kinit -t`.
- Distinguish this from the lesson on Hive itself: schema/locations are catalogue facts; execution is YARN's job.

### 6. Performance and design implications
- Back the metastore with a real, backed-up RDBMS (MySQL/PostgreSQL) — losing it means losing all table definitions even though the data is intact.
- Keep partition counts reasonable (thousands, not millions); every query planning touches metadata.
- Run several HiveServer2 instances behind a load balancer/ZooKeeper discovery for availability.
- Gather stats (`ANALYZE TABLE … COMPUTE STATISTICS`) so the optimizer has real numbers.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hive --service hiveserver2` | Start the JDBC/Thrift server | Multi-user access |
| `hive --service metastore -p 9083` | Start standalone metastore | Remote mode |
| `hive.metastore.uris` | Where clients find the metastore | Client config |
| `hive.server2.thrift.port` | HiveServer2 port (10000) | Firewalls, multiple servers |
| `beeline -u <jdbc-url>` | Connect from CLI | Admin/script access |
| `hive --service help` | List services | Discovery |
| `kinit` / `klist` | Get / inspect a Kerberos ticket | Secured clusters |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Remote metastore (production, shared by Spark/Impala/Presto) | Embedded Derby on any shared system |
| HiveServer2 + Beeline/JDBC for BI tools | Using the legacy Hive CLI in new code |
| Standalone metastore service to isolate DB credentials | Giving every user direct DB access (local mode) |

### 9. Common mistakes
- Embedded Derby in production → "Another instance of Derby may have already booted" and no sharing → move to remote metastore on MySQL/PostgreSQL.
- Not backing up the metastore DB → catalogue loss → regular dumps.
- Pointing Spark at a different metastore than Hive → tables "disappear" → share `hive.metastore.uris`.
- Forgetting `kinit` or using an expired ticket on a secured cluster → GSS exceptions → renew or use a keytab.

### 10. 🎯 Interview kit
**30-second answer:** The metastore stores table metadata — schemas, locations, partitions, formats — in a relational database, exposed by a Thrift service. In production it runs as a remote service shared by Hive, Spark, Impala and Presto, while clients connect through HiveServer2 over JDBC/ODBC.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Three metastore modes?* → embedded, local, remote and what each lets you do.
- *What breaks if the metastore DB is lost?* → data survives in HDFS, definitions don't; recreate as external tables.
- *HiveServer vs HiveServer2?* → authentication and concurrency.
- *What is HCatalog?* → metastore access for Pig/MapReduce.

**Signals interviewers look for:** clear metadata/data separation and awareness of operational concerns (backup, HA, security).

### 11. Check yourself
- [ ] Draw the path from a BI tool to HDFS with HiveServer2 and the metastore.
- [ ] State what each of the three metastore modes shares.
- [ ] List the three Kerberos exchanges.
- [ ] Explain why you'd see a `metastore_db` directory in the wrong place.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Hive › The Metastore
- *Hadoop: The Definitive Guide* — Tom White — Hive › Hive Services
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — S
- *Hadoop: The Definitive Guide* — Tom White — Hive › Hive clients
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Using the Local Machine
- *Hadoop: The Definitive Guide* — Tom White — Hive › Note
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › A reliable configuration service
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Note
- *Hadoop: The Definitive Guide* — Tom White — Spark › Functions
- *Database Internals* — Alex Petrov — Introduction and Overview › Local and Remote Execution

