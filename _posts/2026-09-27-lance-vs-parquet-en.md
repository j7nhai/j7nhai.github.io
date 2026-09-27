---
tags: [lance, parquet, file-format, vector-search, AI]
lang: en
ref: lance-vs-parquet
permalink: /2026/09/27/lance-vs-parquet.html
---

## Lance: The Data Format Built for AI

### What Is Lance

🎯 Lance is an open-source columnar data format designed for ML/AI ([official site](https://lance.org), [GitHub](https://github.com/lance-format/lance)). It sits at the dataset layer of AI infrastructure: training set storage, feature replay, RAG corpora, and embedding retrieval.

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 420" width="100%" style="max-width:640px;height:auto">
<rect width="640" height="420" rx="8" fill="#1e1e2e"/>
<rect x="20" y="20" width="150" height="34" rx="6" fill="#313244" stroke="#89b4fa" stroke-width="1.5"/>
<text x="95" y="42" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">INPUT / DATA</text>
<rect x="20" y="70" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="95" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Data Sources</text>
<rect x="20" y="120" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="145" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Processing</text>
<rect x="20" y="170" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="195" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Feature Eng</text>
<rect x="220" y="20" width="200" height="42" rx="6" fill="#313244" stroke="#f9e2af" stroke-width="1.5"/>
<text x="320" y="46" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">AI INFRASTRUCTURE</text>
<rect x="220" y="75" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="96" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">COMPUTE</text>
<rect x="220" y="118" width="200" height="32" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="2"/>
<text x="320" y="139" text-anchor="middle" fill="#a6e3a1" font-size="13" font-family="monospace">STORAGE · Lance</text>
<rect x="220" y="161" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="182" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">NETWORKING</text>
<rect x="220" y="204" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="225" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">CLOUD</text>
<rect x="220" y="247" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="268" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">MANAGEMENT</text>
<rect x="470" y="20" width="150" height="34" rx="6" fill="#313244" stroke="#cba6f7" stroke-width="1.5"/>
<text x="545" y="42" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">OUTPUT / APP</text>
<rect x="470" y="70" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="92" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">AI Models</text>
<rect x="470" y="115" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="137" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Deployment</text>
<rect x="470" y="160" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="182" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">AI Apps</text>
<rect x="470" y="205" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="227" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Business Value</text>
<line x1="172" y1="140" x2="216" y2="140" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="424" y1="140" x2="466" y2="140" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<text x="50" y="308" fill="#6c7086" font-size="12" font-family="monospace">Data Pipeline: end-to-end on a single Lance dataset</text>
<rect x="30" y="320" width="98" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="79" y="340" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">Ingest</text>
<rect x="148" y="320" width="118" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="207" y="340" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">Clean/Transform</text>
<rect x="286" y="320" width="98" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="335" y="340" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">embedding</text>
<rect x="404" y="320" width="98" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="453" y="340" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">Training</text>
<rect x="522" y="320" width="98" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="571" y="340" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">Search/RAG</text>
<line x1="128" y1="336" x2="144" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="266" y1="336" x2="282" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="384" y1="336" x2="400" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="502" y1="336" x2="518" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<rect x="30" y="362" width="600" height="30" rx="5" fill="none" stroke="#a6e3a1" stroke-width="1.5" stroke-dasharray="4,3"/>
<text x="330" y="382" text-anchor="middle" fill="#a6e3a1" font-size="12" font-family="monospace">One Lance dataset · versioned · no export · no conversion</text>
<text x="320" y="410" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">Scale as needed · Secure by design · High performance · Cost efficient</text>
<defs>
<marker id="ar" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#cdd6f4"/></marker>
<marker id="ar2" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#f38ba8"/></marker>
</defs>
</svg>
</div>

Lance is an open format specification. Its relationship to a dataset: Lance data files + manifest versioning = a Lance dataset.

The entire pipeline can run as a closed loop on a single Lance dataset: ingestion, cleaning, transformation, embedding, training, and retrieval all read and write the same versioned data, with no intermediate export or conversion to other formats. The things ML workloads need — random access, fast updates, vector search — are natively supported at the format level.

### Parquet Refresher

Many of Lance's design choices target Parquet's weak spots, so a quick refresher before we compare.

Columnar storage organized as Row Group → Column Chunk → Page (see the [official Parquet format spec](https://parquet.apache.org/docs/file-format/)). High compression ratio and great scan throughput have made it the de facto standard for data lakes.

The cost is slow random access: even a point lookup must read and decompress an entire page, causing significant read amplification. It is also immutable, unversioned, and has no vector search capability — all of which must be patched with Iceberg/Delta plus a bolted-on FAISS.

There are other ways to patch it. Paimon's primary-key tables wrap parquet files in an LSM-tree: rows are sorted by primary key into SST files, reads merge multiple sorted runs, and compaction merges and deduplicates. The LSM structure at the table layer buys update and point-lookup capability (see the [official Paimon primary-key table docs](https://paimon.apache.org/docs/master/primary-key-table/)). The cost is the extra merge-on-read overhead — and what got fast is the LSM layer, not random reads inside the parquet files themselves, whose read amplification is unchanged.

### The Lance Format

The Lance format has two layers: the table format manages versions, and the file format manages physical storage. First, the core building blocks (see the [official Lance format spec](https://lance.org/format/): [table format](https://lance.org/format/table/) and [file format](https://lance.org/format/file/)):

- fragment: a data segment, the basic unit of append writes, containing multiple data files
- data file: the actual column data files
- delete file: records the positions of deleted/overwritten old rows; deletion is lazy
- manifest: the version manifest, naturally ordered, forming MVCC (Multi-Version Concurrency Control)

Opening a dataset to find the latest version: V2 manifest paths use `max_u64 - version number` as the file name, so they sort naturally — just list and take the head, O(1) (see the [versioning spec](https://lance.org/format/table/versioning/)).

Each fragment has at most one delete file; consecutive deletes are merged into it, and the old one becomes an orphan file.

Side by side:

| Dimension | Parquet | Lance |
|------|---------|-------|
| Updates | Rewrite the whole file | Native update/delete; delete files merged lazily |
| Versioning | None — needs Iceberg/Delta | MVCC built into the format |
| Random reads | Page-level decompression, heavy amplification | Optimized for point lookups |
| Vector search | Not supported | Built-in IVF_PQ, IVF_HNSW and other vector indexes (composable PQ/SQ/RQ quantization) |

### Why Random Reads Are Fast

The answer is written into the file layout: metadata precomputes the "row number → page → byte range" mapping, so reading one row = locate the page + one range read + decode, touching nothing irrelevant.

The sequence for opening a data file (corresponding to `read_all_metadata` in the v2 reader):

1. Read the block at the tail of the file, decode the footer, and get a pointer to the column metadata region
2. Follow the pointer to read the schema and column metadata, yielding the page table for each column
3. Each page's entry in the page table records `num_rows` (row count) and the buffer's byte range

After that, a point lookup is pure addressing: given a row number, binary-search the page table by cumulative row count to find the page, then read only that page's buffer range. v2 pages are not compressed by default (it's optional), the layout is regular, and for fixed-width columns the offset can even be computed directly as "row number × width". This is the mirror image of a Parquet point lookup: Parquet pages use dictionary encoding + compression inside, so rows have no stable mapping to bytes, and you must decompress the whole page before finding anything.

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 310" width="100%" style="max-width:640px;height:auto">
<rect width="640" height="310" rx="8" fill="#1e1e2e"/>
<text x="24" y="32" fill="#a6e3a1" font-size="14" font-weight="bold" font-family="monospace">① Open file (once) · metadata cache</text>
<rect x="24" y="44" width="130" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="89" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">Read tail block</text>
<rect x="176" y="44" width="180" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="266" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">footer → metadata ptr</text>
<rect x="378" y="44" width="200" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="478" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">column meta → page table</text>
<line x1="158" y1="61" x2="172" y2="61" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="360" y1="61" x2="374" y2="61" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<text x="24" y="118" fill="#a6e3a1" font-size="14" font-weight="bold" font-family="monospace">② Point lookup (per row)</text>
<rect x="24" y="130" width="80" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="64" y="151" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">row_id</text>
<rect x="124" y="130" width="190" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="2"/>
<text x="219" y="151" text-anchor="middle" fill="#a6e3a1" font-size="13" font-weight="bold" font-family="monospace">binary search → page k</text>
<rect x="334" y="130" width="110" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="389" y="146" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">PageInfo</text>
<text x="389" y="160" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">buffer range</text>
<rect x="464" y="130" width="150" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="539" y="146" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">read page + decode</text>
<text x="539" y="160" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">→ row</text>
<line x1="108" y1="147" x2="120" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="318" y1="147" x2="330" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="448" y1="147" x2="460" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<path d="M219,172 C219,194 320,202 420,226" stroke="#a6e3a1" stroke-width="1.5" fill="none" stroke-dasharray="4,3" marker-end="url(#pg)"/>
<rect x="84" y="232" width="118" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="143" y="251" text-anchor="middle" fill="#cdd6f4" font-size="11" font-family="monospace">page 0 · 0–8191</text>
<rect x="210" y="232" width="140" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="280" y="251" text-anchor="middle" fill="#cdd6f4" font-size="11" font-family="monospace">page 1 · 8192–16383</text>
<rect x="358" y="232" width="40" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="378" y="251" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">…</text>
<rect x="406" y="232" width="96" height="30" rx="5" fill="#45324a" stroke="#a6e3a1" stroke-width="2"/>
<text x="454" y="251" text-anchor="middle" fill="#a6e3a1" font-size="12" font-weight="bold" font-family="monospace">page k ★</text>
<rect x="510" y="232" width="40" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="530" y="251" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">…</text>
<text x="320" y="282" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">Page table (per column): each page records num_rows + buffer range</text>
<text x="320" y="300" text-anchor="middle" fill="#a6e3a1" font-size="12" font-family="monospace">row number → page → byte range: precomputed in metadata · O(log P)</text>
<defs>
<marker id="pr" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#cdd6f4"/></marker>
<marker id="pg" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#a6e3a1"/></marker>
</defs>
</svg>
</div>

The dataset layer adds one more level of addressing: row id = fragment id + row offset within the fragment, with delete files filtering out deleted rows — the same locating path works across files. Random sampling for training and fetching rows by id for inference both go down this road.

### Concurrent Commits

Versioning inevitably brings concurrency. Lance follows the same approach as lake table formats (Iceberg/Delta/Paimon): old data is untouched until commit, which atomically publishes a new manifest; concurrency is optimistic, and conflicts are adjudicated by operation semantics. An update is just an append: new fragments hold the updated rows, plus a delete file marks the old rows.

The conflict matrix:

| Operation | Physical action | Conflicts |
|------|----------|------|
| append | new fragment + new manifest | No conflict; automatic rebase and retry |
| delete | new delete file, original data untouched | No conflict with append; conflicts with overwrite |
| update | new fragment with updated rows + delete file on old rows | Same as delete |
| overwrite | drop old fragments, write new fragments | Conflicts with everything; fails immediately |
| merge_insert | upsert: matched rows update, new rows insert | Bloom Filter detects key conflicts; retries up to N times, then fails with `TooMuchWriteContention` |
| create_index | write index file + new manifest | No conflict with append/delete; conflicts with overwrite |

### Hybrid Search

Real-world queries rarely use "semantically similar" as the only condition; they usually combine "semantically similar + scalar filter", e.g. "find the top-k nearest to the query vector, where age > 30 and category = news". This is hybrid search (also called filtered vector search): vector ANN search and scalar filtering (BTree and similar indexes) work together within a single query. Vector search is Lance's most fundamental capability gain over Parquet: the index is part of the format, not a bolted-on service. An index is a combination of "IVF clustering + sub-index + quantization" (see the [Lance vector index spec](https://lance.org/format/index/vector/)): sub-indexes include FLAT/HNSW, quantization includes PQ/SQ/RQ, yielding combinations like IVF_PQ and IVF_HNSW_PQ. Below we use the most common, IVF_PQ, as the example.

PQ (Product Quantization) is compression + approximate distance, not a metric:

- Split the vector into N sub-vectors; run k-means on each to get a 256-centroid codebook
- Storage keeps only one code (1 byte) per sub-vector

```
128-dim float32 = 512B → 16 codes = 16B, a 32× compression
```

At index build time: within each cluster, vectors first subtract the centroid to compute residuals, which are then quantized. The index file stores three things: the IVF centroids, the PQ codebooks (both in the footer's global buffers), and the per-vector codes — only the original vectors are no longer stored.

At query time: precompute lookup tables from the codebooks; each candidate's distance = 16 table lookups summed, without reading original vectors.

Scale estimation:

| Scale | PQ codes | row_ids | Total |
|------|---------:|--------:|-----:|
| 1 billion | 640 GB | 80 GB | ~720 GB |
| 10 billion | 6.4 TB | 800 GB | ~7.2 TB |

The query plan for a prefilter hybrid search:

1. The BTree runs first, producing the set of row_addrs that satisfy the filter
2. The set is wrapped into a RowAddrMask (allow_list) passed into the vector search
3. While scanning PQ codes, each row is checked against the mask and skipped if absent
4. The top-k therefore satisfies the filter by construction

### How to Choose

No format is a universal answer; choose by workload.

For analytics on the lake, use Parquet — its ecosystem is irreplaceable. For streaming upserts and CDC ingestion, use Paimon — the high-frequency updates its LSM structure buys are its home turf. For ML training sets and RAG corpora (random sampling + frequent versioning + vector search), use Lance. They can also coexist: keep detail data in the Parquet lake, and put embeddings in Lance.
