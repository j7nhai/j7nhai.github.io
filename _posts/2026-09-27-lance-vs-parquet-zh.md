---
tags: [lance, parquet, file-format, vector-search, AI]
lang: zh
ref: lance-vs-parquet
permalink: /zh/2026/09/27/lance-vs-parquet.html
---

## Lance：为 AI 而生的数据格式

### Lance 是什么

🎯 Lance 是为 ML/AI 设计的开源列式数据格式（[官网](https://lance.org)、[GitHub](https://github.com/lance-format/lance)），定位在 AI infra 中的 dataset 层：训练集存储、特征回放、RAG 语料、embedding 检索。

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 420" width="640" height="420">
<rect width="640" height="420" rx="8" fill="#1e1e2e"/>
<rect x="20" y="20" width="150" height="34" rx="6" fill="#313244" stroke="#89b4fa" stroke-width="1.5"/>
<text x="95" y="42" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">INPUT / DATA</text>
<rect x="20" y="70" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="95" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">数据源</text>
<rect x="20" y="120" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="145" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">数据处理</text>
<rect x="20" y="170" width="150" height="40" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="95" y="195" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">特征工程</text>
<rect x="220" y="20" width="200" height="42" rx="6" fill="#313244" stroke="#f9e2af" stroke-width="1.5"/>
<text x="320" y="46" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">AI INFRASTRUCTURE</text>
<rect x="220" y="75" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="96" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">COMPUTE 计算</text>
<rect x="220" y="118" width="200" height="32" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="2"/>
<text x="320" y="139" text-anchor="middle" fill="#a6e3a1" font-size="13" font-family="monospace">STORAGE 存储 · Lance</text>
<rect x="220" y="161" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="182" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">NETWORKING 网络</text>
<rect x="220" y="204" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="225" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">CLOUD 云</text>
<rect x="220" y="247" width="200" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="268" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">MANAGEMENT 管理</text>
<rect x="470" y="20" width="150" height="34" rx="6" fill="#313244" stroke="#cba6f7" stroke-width="1.5"/>
<text x="545" y="42" text-anchor="middle" fill="#cdd6f4" font-size="14" font-weight="bold" font-family="monospace">OUTPUT / APP</text>
<rect x="470" y="70" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="92" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">AI 模型</text>
<rect x="470" y="115" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="137" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">部署</text>
<rect x="470" y="160" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="182" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">AI 应用</text>
<rect x="470" y="205" width="150" height="35" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="545" y="227" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">业务价值</text>
<line x1="172" y1="140" x2="216" y2="140" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="424" y1="140" x2="466" y2="140" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<text x="50" y="308" fill="#6c7086" font-size="12" font-family="monospace">数据 Pipeline：全程在 Lance dataset 上完成</text>
<rect x="30" y="320" width="104" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="82" y="340" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">数据摄取</text>
<rect x="154" y="320" width="104" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="206" y="340" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">清洗/转换</text>
<rect x="278" y="320" width="104" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="330" y="340" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">embedding</text>
<rect x="402" y="320" width="104" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="454" y="340" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">训练</text>
<rect x="526" y="320" width="104" height="32" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="578" y="340" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">检索/RAG</text>
<line x1="138" y1="336" x2="150" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="262" y1="336" x2="274" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="386" y1="336" x2="398" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<line x1="510" y1="336" x2="522" y2="336" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#ar)"/>
<rect x="30" y="362" width="600" height="30" rx="5" fill="none" stroke="#a6e3a1" stroke-width="1.5" stroke-dasharray="4,3"/>
<text x="330" y="382" text-anchor="middle" fill="#a6e3a1" font-size="12" font-family="monospace">同一个 Lance dataset · 版本化 · 免导出 · 免转换</text>
<text x="320" y="410" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">按需扩展 · 天生安全 · 高性能 · 成本高效</text>
<defs>
<marker id="ar" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#cdd6f4"/></marker>
<marker id="ar2" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#f38ba8"/></marker>
</defs>
</svg>
</div>

Lance 是一套开放格式规范，它和 dataset 的关系：Lance 数据文件 + manifest 版本管理 = Lance dataset。

整个 pipeline 可以在一个 Lance dataset 上闭环完成：摄取、清洗、转换、embedding、训练、检索读写同一份版本化数据，中途不需要导出或转换成其它格式。ML 场景要的随机读、快速更新、向量检索，Lance 在格式层原生支持。

### Parquet 回顾

Lance 的很多设计都是对着 Parquet 的短板来的，比较前先快速回顾它。

列式存储，按 Row Group → Column Chunk → Page 组织（详见 [Parquet 官方格式规范](https://parquet.apache.org/docs/file-format/)）。压缩率高、扫描吞吐好，是数据湖事实标准。

代价是随机访问慢：点查也要读整页并解压，读放大显著。且不可更新、无版本、无向量检索能力，这些都要靠 Iceberg/Delta 和外挂 FAISS 补齐。

也有别的补法。Paimon 主键表用 LSM-tree 包装 parquet 文件：按主键排序写 SST 文件，读取时归并多个 sorted run，compaction 归并去重，靠表层的 LSM 结构换来了更新和点查能力（详见 [Paimon 官方主键表文档](https://paimon.apache.org/docs/master/primary-key-table/)）。代价是 merge-on-read 的额外合并成本，而且快的是 LSM 这一层，parquet 文件内部的随机读放大并没有改变。

### Lance 格式

Lance 格式分两层：表格式管版本，文件格式管物理存储。先看核心单元（详见 [Lance 官方格式规范](https://lance.org/format/)：[表格式](https://lance.org/format/table/) 和 [文件格式](https://lance.org/format/file/)）：

- fragment：数据片段，追加式写入的基本单位，内含多个 data file
- data file：实际的列数据文件
- delete file：记录被删除/被更新旧行的位置，删除是惰性的
- manifest：版本清单，天然有序，构成 MVCC（Multi-Version Concurrency Control，多版本并发控制）

打开 dataset 找最新版本：V2 manifest paths，文件名为 `max_u64 - 版本号`，天然有序，list 取 head 即可，O(1)（详见 [版本管理规范](https://lance.org/format/table/versioning/)）。

delete file 一个 fragment 只有一个，连续 delete 会合并，旧的成为孤儿文件。

对照：

| 维度 | Parquet | Lance |
|------|---------|-------|
| 更新 | 重写整个文件 | 原生 update/delete，delete file 惰性合并 |
| 版本 | 无，靠 Iceberg/Delta | 格式内置 MVCC |
| 随机读 | 页级解压，放大明显 | 针对点查优化 |
| 向量检索 | 不支持 | 内置 IVF_PQ、IVF_HNSW 等向量索引（可组合 PQ/SQ/RQ 量化） |

### 为什么随机读快

答案写在文件布局里：元数据把「行号 → 页 → 字节区间」的映射提前算好了，读一行 = 定位页 + 一次 range read + 解码，不碰无关数据。

打开一个 data file 的顺序（对应 v2 reader 的 `read_all_metadata`）：

1. 读文件尾部一个 block，解出 footer，拿到列元数据区的指针
2. 顺着指针读 schema 和列元数据，得到每列的页表
3. 页表里每个页记录了 `num_rows`（行数）和 buffer 的字节区间

之后的点查就是纯定位：给定行号，在页表上按行号二分找到所在页，只读该页的 buffer 区间。v2 页默认不做整页压缩（可选），布局规整，定宽列甚至能按「行号 × 宽度」直接算出偏移。这是 Parquet 点查的镜像：Parquet 页内字典编码 + 压缩，行与字节没有稳定映射，只能整页解压再找。

<div align="center">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 300" width="100%" style="max-width:640px;height:auto">
<rect width="640" height="300" rx="8" fill="#1e1e2e"/>
<text x="24" y="32" fill="#a6e3a1" font-size="14" font-weight="bold" font-family="monospace">① 打开文件（一次）· 元数据缓存</text>
<rect x="24" y="44" width="120" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="84" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">读文件尾部 tail</text>
<rect x="176" y="44" width="150" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="251" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">footer → 元数据指针</text>
<rect x="358" y="44" width="160" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="438" y="65" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">列元数据 → 页表</text>
<line x1="148" y1="61" x2="172" y2="61" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="330" y1="61" x2="354" y2="61" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<text x="24" y="118" fill="#a6e3a1" font-size="14" font-weight="bold" font-family="monospace">② 点查（每次）</text>
<rect x="24" y="130" width="92" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="70" y="151" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">行号 row_id</text>
<rect x="148" y="130" width="150" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="2"/>
<text x="223" y="151" text-anchor="middle" fill="#a6e3a1" font-size="13" font-weight="bold" font-family="monospace">页表二分 → 页 k</text>
<rect x="322" y="130" width="136" height="34" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="390" y="146" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">PageInfo</text>
<text x="390" y="160" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">buffer 区间</text>
<rect x="480" y="130" width="140" height="34" rx="5" fill="#313244" stroke="#a6e3a1" stroke-width="1.5"/>
<text x="550" y="151" text-anchor="middle" fill="#cdd6f4" font-size="13" font-family="monospace">读单页 + 解码 → 行</text>
<line x1="120" y1="147" x2="144" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="302" y1="147" x2="318" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<line x1="462" y1="147" x2="476" y2="147" stroke="#cdd6f4" stroke-width="1.5" marker-end="url(#pr)"/>
<path d="M223,170 C223,190 340,186 428,196" stroke="#a6e3a1" stroke-width="1.5" fill="none" stroke-dasharray="4,3" marker-end="url(#pg)"/>
<text x="24" y="204" fill="#6c7086" font-size="12" font-family="monospace">页表（每列）：每页记录 num_rows + buffer 区间，按累计行数二分</text>
<rect x="24" y="216" width="112" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="80" y="235" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">页0 行 0–8191</text>
<rect x="144" y="216" width="112" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="200" y="235" text-anchor="middle" fill="#cdd6f4" font-size="12" font-family="monospace">页1 8192–16383</text>
<rect x="264" y="216" width="112" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="320" y="235" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">…</text>
<rect x="384" y="216" width="112" height="30" rx="5" fill="#45324a" stroke="#a6e3a1" stroke-width="2"/>
<text x="440" y="235" text-anchor="middle" fill="#a6e3a1" font-size="12" font-weight="bold" font-family="monospace">页k ★</text>
<rect x="504" y="216" width="112" height="30" rx="5" fill="#313244" stroke="#585b70" stroke-width="1"/>
<text x="560" y="235" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">…</text>
<text x="440" y="268" text-anchor="middle" fill="#a6e3a1" font-size="12" font-family="monospace">二分 O(log P)，P = 页数</text>
<text x="320" y="292" text-anchor="middle" fill="#6c7086" font-size="12" font-family="monospace">行号 → 页 → 字节区间，映射在元数据里提前算好</text>
<defs>
<marker id="pr" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#cdd6f4"/></marker>
<marker id="pg" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto"><path d="M0,0 L8,3 L0,6Z" fill="#a6e3a1"/></marker>
</defs>
</svg>
</div>

dataset 层再加一层寻址：row id = fragment id + fragment 内行偏移，配合 delete file 过滤被删行，跨文件同样定位。训练随机抽样、推理按 id 取数，走的都是这条路。

### 并发提交

版本化必然带来并发问题。Lance 这套和湖表格式（Iceberg/Delta/Paimon）是同一个思路：写前不动旧数据，commit 时原子发布新 manifest，乐观并发，冲突按操作语义判定。更新即 append：新 fragment 写更新行 + 旧行写 delete file。

冲突矩阵：

| 操作 | 物理动作 | 冲突 |
|------|----------|------|
| append | 新 fragment + 新 manifest | 不冲突，自动 rebase 重试 |
| delete | 新 delete file，原数据不动 | 与 append 不冲突，与 overwrite 冲突 |
| update | 新 fragment 写更新行 + 旧行 delete file | 同 delete |
| overwrite | 删旧 fragment，写新 fragment | 与所有操作冲突，直接报错 |
| merge_insert | upsert：匹配行 update，新行 insert | Bloom Filter 检测 key 冲突，可重试 N 次，超限报 `TooMuchWriteContention` |
| create_index | 写索引文件 + 新 manifest | 与 append/delete 不冲突，与 overwrite 冲突 |

### 混合检索

实际查询很少只用「语义相近」一个条件，通常是「语义相近 + 标量过滤」的组合，比如「找与 query 向量最近的 top-k，且 age > 30、类目 = 新闻」。这就是混合检索（hybrid search / filtered vector search）：向量 ANN 检索和标量过滤（BTree 等索引）在一次查询里协同完成。向量检索是 Lance 相对 Parquet 最本质的增量能力：索引是格式的一部分，不是外挂服务。索引是「IVF 聚类 + 子索引 + 量化」的组合（[Lance 向量索引规范](https://lance.org/format/index/vector/)）：子索引有 FLAT/HNSW，量化有 PQ/SQ/RQ，组合出 IVF_PQ、IVF_HNSW_PQ 等，下面以最常用的 IVF_PQ 为例。

PQ（乘积量化）是压缩 + 近似距离，不是度量：

- 向量切 N 段，每段 k-means 出 256 个中心的 codebook
- 存储只存每段 code（1 字节）

```
128维 float32 = 512B → 16 个 code = 16B，32 倍压缩
```

建索引时：簇内向量先减质心算 residual，再量化。索引文件存三样：IVF 质心、PQ codebook（都在 footer 的 global buffers 里）和每条向量的 code，只是原始向量不再存。

查询时：用 codebook 预计算查找表，每条候选距离 = 查 16 次表累加，不读原始向量。

规模估算：

| 规模 | PQ codes | row_ids | 总计 |
|------|---------:|--------:|-----:|
| 10 亿 | 640 GB | 80 GB | ~720 GB |
| 100 亿 | 6.4 TB | 800 GB | ~7.2 TB |

混合检索的 prefilter 查询计划：

1. BTree 先执行，输出满足条件的 row_addr 集合
2. 封装成 RowAddrMask（allow_list）传入向量搜索
3. 扫 PQ codes 时逐条查 mask，不在则跳过
4. top-k 天然满足 filter

### 怎么选

格式没有万能解，按场景选。

湖上分析用 Parquet，生态不可替代。流式 upsert、CDC 入湖用 Paimon，LSM 换来的高频更新是它的主场。ML 训练集、RAG 语料（随机抽样 + 频繁版本 + 向量检索）用 Lance。也可共存：明细留 Parquet 湖，embedding 用 Lance。
