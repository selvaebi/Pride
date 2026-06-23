# PRIDE

Interview prep notes: using the **PRIDE Archive** (PRoteomics IDEntifications, EMBL-EBI) as a real-world case study for **Microsoft / Copilot-scale distributed systems** interviews — STAR stories, system-design talking points, and what to say as a software developer.

> Honesty note: STAR stories must reflect *your own* work. The stories below are built around PRIDE's *real* engineering challenges (from the published papers). Adapt them to what you actually did, or use them as "here's how I'd reason about a system like PRIDE" system-design talking points.

---

## The PRIDE facts that matter in an interview

PRIDE is the world's largest mass-spectrometry proteomics repository. Concrete numbers separate a strong answer from a vague one:

- **Storage:** grew from **1.35 PB (Mar 2021) → 285 PB (Aug 2024)**. Third-largest omics archive at EMBL-EBI.
- **Datasets:** **42,036 datasets** (Aug 2024), ~**534 new datasets/month**, 44.9% submitted in the last 3 years. One submission held **7,444 raw files / >15,000 total files**.
- **Traffic:** **>100 TB downloaded per month**.
- **Architecture:** full **microservice architecture** on **Kubernetes**, each API scaled independently; **MongoDB + Apache Solr** for search/indexing, every storage item **sharded and replicated across two EMBL-EBI datacenters** for reliability.
- **Pipelines:** **Apache Spark/PySpark** to group **millions of PSMs in <6 hours**; an **NLP-based validation pipeline that cut validation from 34 hours → 4 minutes**; a **granular resubmission pipeline** (re-validate only changed files); file **checksums computed at submission**.
- **Transfer:** FTP, Aspera, and newly **Globus** for asynchronous transfer of huge datasets without connection timeouts; a **chunk-based streaming API** that never loads full files into memory.

---

## STAR stories

### 1. Scaling the search/index tier independently (core "distributed scaling" story)

- **S** — PRIDE's monolithic indexing couldn't keep up: 42K datasets, ~534 new/month, search queries spanning peptides/proteins across hundreds of millions of records. Search latency degraded under download spikes (>100 TB/month).
- **T** — Decouple search from storage and ingestion so each scales to its own load profile, without one noisy workload starving another.
- **A** — Moved to a microservice architecture on Kubernetes. Split into independent APIs (archive query, file streaming, USI spectrum retrieval). Backed search with **Apache Solr sharded across two datacenters**, MongoDB as the document store. Each API became horizontally scalable by replica count; the streaming API processed files in chunks instead of loading them into memory.
- **R** — Each tier scales on its own metric (query QPS vs. ingestion throughput vs. egress bandwidth), datacenter-level redundancy gives failover, and search no longer collapses during download bursts.

**Copilot mapping:** Separate read/serving path from write/ingestion path, shard the index, autoscale stateless services on K8s — exactly the pattern for a code-search or completion backend.

### 2. Batch-processing at scale with Spark (data-pipeline story)

- **S** — A single submission could contain millions of peptide-spectrum matches (PSMs); grouping/aggregating on one machine was too slow to keep monthly intake flowing.
- **T** — Process millions of PSMs in a bounded time window as part of post-submission processing.
- **A** — Implemented **sparkMS on Spark/PySpark**, partitioning PSMs and doing distributed group-by aggregation, then bulk-indexing results into Solr/MongoDB.
- **R** — Grouped **millions of PSMs in under 6 hours**, making large-dataset analysis feasible at archive scale.

**Copilot mapping:** Distributed map/shuffle/reduce, partitioning strategy, handling skew — same vocabulary as large-scale offline jobs (training-data prep, telemetry aggregation).

### 3. Cutting validation latency 500× with automation (reliability story)

- **S** — Dataset validation was largely manual and took **~34 hours**, a bottleneck as submissions grew.
- **T** — Automate validation without sacrificing data integrity.
- **A** — Built a rules + **NLP pipeline** to auto-validate metadata and submissions; added **checksums at submission time** validated downstream to guarantee integrity.
- **R** — Validation dropped from **34 hours to 4 minutes**, removing the human bottleneck while keeping correctness guarantees.

### 4. Huge-file transfer without timeouts (Globus / resilience story)

- **S** — Submissions of >15,000 files / multi-TB hit firewall blocks (Aspera) and connection timeouts (FTP).
- **T** — Reliable transfer of very large datasets across unreliable institutional networks.
- **A** — Integrated **Globus for asynchronous, resumable transfers**; built a **granular resubmission pipeline** that re-validates only modified files instead of re-uploading entire datasets.
- **R** — Eliminated timeout failures on the largest submissions and made updates incremental rather than all-or-nothing.

**Copilot mapping:** Idempotency, resumability, partial/delta updates, backpressure — directly relevant to large-artifact handling in distributed systems.

---

## System-design talking points ("design a system like this")

Use PRIDE as the worked example for classic distributed-systems themes:

- **Scalability vs. reliability as explicit drivers** — PRIDE states these two guided the new architecture: shard + replicate every storage item across two datacenters.
- **Polyglot persistence** — MongoDB (document store) + Solr (inverted index) instead of one DB doing both jobs.
- **Independent horizontal scaling** — stateless APIs on Kubernetes, scaled by replica count per workload.
- **Streaming over buffering** — chunked file API keeps memory bounded regardless of file size.
- **Batch vs. serving separation** — Spark for heavy offline aggregation, fast APIs for the serving path.
- **Trade-offs to volunteer** — strong consistency isn't needed for an append-mostly archive → favor availability/partition tolerance; checksums give integrity without distributed transactions.

---

## What to say as a software developer

> "PRIDE is a useful real-world case study in scaling an append-heavy data system: it went from ~1 PB to 285 PB while keeping search fast by separating concerns — a Solr index sharded across two datacenters for queries, MongoDB for documents, Spark for the heavy offline aggregation, and stateless microservices on Kubernetes that each scale on their own load. The lessons that generalize to Copilot-scale systems: split read/write paths, shard your index, make ingestion idempotent and resumable, and push automation into the validation path — they took validation from 34 hours to 4 minutes."

If the interviewer probes, pivot to trade-offs (consistency model, shard-key choice, hot-partition/skew handling, failover semantics) — that's where senior signal lives.

---

## Sources

- [PRIDE database at 20 years: 2025 update — *Nucleic Acids Research*](https://academic.oup.com/nar/article/53/D1/D543/7874848) ([open-access PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11701690/))
- [The PRIDE database resources in 2022 — *Nucleic Acids Research*](https://pmc.ncbi.nlm.nih.gov/articles/PMC8728295/)
- [PRIDE-Archive/pride-resources (GitHub)](https://github.com/PRIDE-Archive/pride-resources)
