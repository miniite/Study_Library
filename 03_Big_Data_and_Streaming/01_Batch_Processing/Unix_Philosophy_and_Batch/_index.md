---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/Unix_Philosophy_and_Batch"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Unix Philosophy and Batch Processing

**L4 Advanced** · [↑ Batch Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Unix tools — small programs doing one thing, joined by pipes over a uniform byte-stream interface, with immutable inputs — are a miniature version of the design that distributed batch systems like MapReduce inherited.

### 1. The big picture

Before reaching for a cluster, it is worth seeing how much a single machine and a few shell tools can do, and why they are so composable. The Unix philosophy (1978) — do one thing well, make output usable as someone else's input, build and discard quickly, prefer tools to unskilled help — is the conceptual ancestor of MapReduce, Spark and modern pipeline design. Its biggest limit is that it runs on one machine; that is exactly where Hadoop comes in.

This lesson also sets up two ideas used throughout the area: **sorting** as an efficient, disk-friendly way to group data, and **separation of logic from wiring** (a program reads `stdin` and writes `stdout` and does not care where they lead).

### 2. Core concepts and vocabulary

- **Pipe (`|`)** — connects one process's `stdout` to another's `stdin` through a small in-memory buffer, without writing the full intermediate stream to disk.
- **Uniform interface** — in Unix, a file (file descriptor): an ordered sequence of bytes. Files, sockets, terminals and devices share it.
- **Record separator** — by convention `\n`; fields are split by whitespace, tabs, commas, etc.
- **stdin / stdout** — default input and output streams that a shell can wire to anything.
- **Working set** — the amount of memory a job needs random access to; for counting URLs, it depends on the number of *distinct* URLs, not the number of log lines.
- **External sort** — sorting data larger than memory by sorting chunks, spilling them to disk and merging them (mergesort, sequential I/O).
- **Loose coupling / late binding / inversion of control** — the idea that wiring is decided outside the program.
- **Immutable input** — commands do not modify their input files, so experiments are safe.

### 3. How it works, step by step

Finding the five most requested pages in an nginx access log:

```bash
cat /var/log/nginx/access.log |
  awk '{print $7}' |   # 1. keep only the 7th field: the requested URL
  sort             |   # 2. sort so identical URLs become adjacent
  uniq -c          |   # 3. collapse adjacent duplicates, prefix with a count
  sort -r -n       |   # 4. sort numerically by count, largest first
  head -n 5            # 5. keep the first five lines
```

1. `cat` streams the file.
2. `awk` splits each line on whitespace and prints field 7.
3. `sort` makes equal URLs consecutive (spilling to disk if needed).
4. `uniq -c` counts adjacent identical lines.
5. The second `sort` ranks by count; `head` truncates.

| Approach | Memory use | Strength | Weakness |
|---|---|---|---|
| Hash table (Ruby/Python script) | Proportional to distinct keys | Fast when the working set fits in RAM | Fails when distinct keys exceed memory |
| Sort-based pipeline | Chunked; spills to disk | Scales beyond memory with sequential I/O | Extra sort work |

### 4. Worked example

Equivalent custom program (Ruby) from the same task:

```ruby
counts = Hash.new(0)
File.open('/var/log/nginx/access.log') do |file|
  file.each { |line| counts[line.split[6]] += 1 }
end
counts.map { |url, c| [c, url] }.sort.reverse[0...5]
      .each { |c, url| puts "#{c} #{url}" }
```

Both produce, for example:

```text
4189 /favicon.ico
3631 /2013/05/24/improving-security-of-ssh-private-keys.html
2124 /2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html
1369 /
 915 /css/typography.css
```

Modifying the analysis is trivial: exclude stylesheets with `awk '$7 !~ /\.css$/ {print $7}'`, or count visitors instead with `awk '{print $1}'`. Another example from Hadoop's guide: a bash loop that runs `awk` to find the maximum temperature per year across a century of weather files took 42 minutes on one large EC2 instance; to speed up you must split work evenly (file sizes vary widely, so per-year processes finish at different times), combine partial results, and eventually go beyond one machine — which motivates MapReduce.

### 5. Under the hood

**Why composition works.** All programs agree on bytes with newline-separated records, so any program's output can feed any other's input. That agreement is crude (`$7` is not readable, parsing varies) but remarkably effective; few modern systems interoperate as smoothly, and even databases with the same data model rarely make it easy to move data between them ("Balkanisation" of data).

**Separation of logic and wiring.** A well-behaved tool does not open fixed paths; it uses `stdin` and `stdout`, and the shell connects them. Programs with multiple inputs/outputs, or that open network connections themselves, lose this flexibility. You can drop your own program (IP-to-country lookup, user-agent parser) into a pipeline as long as it follows the convention.

**Sorting as the engine.** GNU `sort` automatically handles larger-than-memory input by sorting chunks, writing them as sorted segments, and merging them — the same pattern as SSTables and LSM-trees — and parallelises across cores. The pipeline's bottleneck is typically reading the input from disk.

**Transparency.** Because inputs are immutable you can rerun freely; you can cut the pipeline at any stage to look at output; and you can save an intermediate file and restart from there. MapReduce keeps all three properties (immutable inputs, inspectable intermediate directories, restartable stages).

### 6. Performance and design implications

- Choose hash aggregation when distinct keys fit in memory; choose sort-based grouping when they don't.
- Sequential I/O wins: sort-and-merge is disk-friendly; random access is not.
- Keep stages simple and composable; you can swap one stage without touching others.
- A single machine goes surprisingly far (gigabytes in seconds); only distribute when data or time requirements demand it.
- When designing pipelines (even distributed ones), prefer immutable intermediate datasets and explicit stage boundaries.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `awk '{print $N}'` | Extracts fields | Parsing log or delimited records |
| `sort [-n -r -k -S -T]` | Sorts (numeric, reverse, key; memory/tmp options) | Grouping and ranking |
| `uniq -c` | Counts adjacent duplicates | After `sort` |
| `grep`, `sed`, `cut`, `xargs` | Filter, rewrite, split, fan out | Pipeline stages |
| `head`, `tail`, `less` | Inspect or truncate | Debugging mid-pipeline |
| `tee file` | Save an intermediate stage | Restartable experiments |
| `sort --parallel=N -S 4G` | Parallel, bounded-memory sort | Large files |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Ad-hoc analysis of files up to tens of GB | Data spans machines, or you need fault tolerance |
| Quick prototypes and one-off checks | Production pipelines needing schemas, retries, lineage |
| Hash-table aggregation with a small working set | Number of distinct keys approaches memory |
| Text pipelines with simple formats | Binary/nested data needing typed schemas |

### 9. Common mistakes

- Forgetting to `sort` before `uniq -c` → duplicates counted separately → always sort first.
- Locale-dependent sorting → surprising order → set `LC_ALL=C` for byte order.
- Parsing with whitespace when fields contain spaces → misaligned columns → use a real delimiter or parser.
- Overwriting the input with redirection (`> same_file`) → data loss → write to a new file.
- Assuming it scales to a cluster → single-machine bottleneck → move to Hadoop/Spark once it no longer fits.

### 10. 🎯 Interview kit

**30-second answer:** The Unix philosophy is small programs that do one thing well, connected by pipes over a uniform byte-stream interface, with immutable input and the wiring separated from the logic. MapReduce applies the same ideas to many machines: map ≈ `awk`, shuffle ≈ `sort`, reduce ≈ `uniq -c`.

**Structure a longer answer:** describe the pipeline → explain uniform interface and stdin/stdout → compare sorting vs hash aggregation → highlight single-machine limits → connect to MapReduce.

**Likely follow-ups:**
- *Sorting vs hash table for counting?* → Hash is faster if distinct keys fit in RAM; sorting handles larger-than-memory by external merge.
- *Why are pipes efficient?* → Small in-memory buffer; stages run concurrently without writing the full intermediate.
- *What are Unix tools' limits?* → One machine, text conventions, awkward multi-input/output.

**Signals interviewers look for:** ability to map a pipeline to map/shuffle/reduce and to explain working set vs input size.

### 11. Check yourself

- [ ] Explain each stage of the top-5-URLs pipeline and what would change to count client IPs.
- [ ] Describe the memory behaviour of the hash approach vs the sort approach.
- [ ] State three properties that make Unix experimentation safe.
- [ ] Map `awk | sort | uniq -c` to mapper, shuffle and reducer.
- [ ] Identify two cases where stdin/stdout wiring is insufficient.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy › Separation of logic and wiring
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing › Batch Processing with Unix Tools
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy › Transparency and experimentation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing › Sorting versus in-memory aggregation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing › Chain of commands versus custom program
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Analyzing the Data with Unix Tools
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy › A uniform interface
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing › Simple Log Analysis

