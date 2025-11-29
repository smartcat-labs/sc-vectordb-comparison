# Weaviate Review

## Overview

Weaviate is an open-source vector database designed with a cloud-native architecture and GraphQL/REST/gRPC APIs, focusing on storing both data objects and their vector embeddings. It distinguishes itself through modular, pluggable vectorization, strong hybrid (keyword + vector) search, and native multi-tenancy. It can be run as **Weaviate Cloud** (Shared/Dedicated) or self-hosted on Kubernetes/Docker, giving teams flexibility across MVPs and large-scale deployments.

Since late 2025, Weaviate Cloud uses a **new pricing model** based on **vector dimensions, storage, and backups**, with three plans (Flex, Plus, Premium) replacing the older Standard/Professional/Business Critical tiers and AIU-based Enterprise pricing.

## Architecture

### Core Architecture

- **Collections**: Named sets of data objects and their vectors sharing the same dimensionality, distance metric, and index/compression settings; collections can be single-tenant or multi-tenant.
- **Segmented Storage**: Data is partitioned into shards and segments; each shard holds one tenant in multi-tenant setups, providing isolation and efficient querying.
- **GraphQL API**: Primary interface for complex queries and traversals, with REST and gRPC available for ingestion and high-throughput use cases.
- **Modular Vectorization**: Pluggable modules and integrations for popular embedding providers (OpenAI, Cohere, etc.), plus Weaviate’s own embedding service.
- **Distributed System**: Horizontal scaling via sharding and replication, backed by Raft-based metadata consensus.

### Key Features

- Flexible vectorization with built-in modules or external pipelines.
- Native hybrid search combining semantic vector search with BM25 keyword search.
- Multi-vector and multimodal support (multiple vector fields per object).
- Real-time CRUD with full update support.
- **ACORN-based filtering** as the default strategy (v1.34+), significantly improving filtered search performance on large datasets.

## Pricing Models

### Serverless Cloud Pricing

> Former “Serverless Cloud” with Standard/Professional/Business Critical tiers is now Shared Cloud with three plans: Flex, Plus, Premium, all billed on vector dimensions + storage (GiB) + backups (GiB). (weaviate.io)
> 

**Flex (Shared Cloud, pay-as-you-go):**

- Typical base: **≈ $45/month** (varies by region).
- Vector dimensions: **≈ $0.000327 per 1M dimensions/month** (reference rate from internal 2025 docs).
- Storage: **≈ $0.2125 per GiB/month** (persistent data).
- Backups: **≈ $0.022 per GiB/month**, daily backups with 7-day retention for new subscriptions.
- Designed for evaluation and small production workloads (99.5%–99.9% SLA depending on configuration).

**Plus (Shared or small Dedicated):**

- Typical base: **≈ $280/month**.
- Uses the same unit metrics (vector dimensions, storage, backups), often at similar or discounted rates relative to Flex, depending on contract.
- Daily backups stored for **30 days**.
- Higher SLA (up to 99.9%) and stronger support options.

**Premium (Dedicated Cloud / Enterprise):**

- Annual contracts with higher base commitments (low five figures/year and up).
- Dedicated infrastructure (single-tenant), 99.9%–99.95% SLA, private networking, and compliance options (SOC 2, HIPAA).
- Same *pricing dimensions* (vector dimensions, storage, backups) but with enterprise discounts and custom sizing.

**Example Cost (1M products, 768-dim embeddings):**

Assume:

- 1M product embeddings at 768 dimensions → **768M vector dimensions**.
- ~2.9 GiB of total data (vectors + indexes + metadata).
- Single replica (for simplicity; HA would scale dimensions and storage proportionally).

**Flex (Shared Cloud):**

- Vector dimensions: 768M × $0.000327 ≈ **$0.25/month**
- Storage: 2.9 GiB × $0.2125 ≈ **$0.62/month**
- Backups: 2.9 GiB × $0.022 ≈ **$0.06/month**
- Base fee: **≈ $45/month**

**Total Flex:** ~**$46/month**

**Plus (Shared/Dedicated):**

- Same usage charges (~$0.25 + $0.62 + $0.06 ≈ $0.93/month)
- Base fee: **≈ $280/month**

**Total Plus:** ~**$281/month**

> Older pricing references like $0.095 / 1M dims with a $25/month “Standard” tier and separate HA multipliers are now deprecated and replaced by the above model. (Weaviate Community Forum)
> 

### Enterprise Cloud Pricing

Previously, Enterprise Cloud used an **AI Unit (AIU)** system with HOT/WARM/COLD storage tiers and a 3× multiplier for HA. That model has been **retired**.

Today’s **Dedicated Cloud (Premium / some Plus contracts)**:

- Uses the **same metrics** as Shared Cloud: vector dimensions, storage, backups.
- Adds a **contracted base commit** (e.g., starting around $10k/year on marketplaces) plus overage charges per 1M dimensions.
- Makes most sense when you consolidate multiple workloads and need SLAs, security, and isolation that Shared Cloud cannot provide, rather than for a single small index.

## Performance Characteristics

| Configuration | QPS (approx) | P99 Latency (typical) | Recall (typical) | Use Case |
| --- | --- | --- | --- | --- |
| Shared Cloud (Flex/Plus) | 100s–low 1000s | ~50–200ms (warm) | 0.80–0.97 | Small–mid production / RAG |
| Dedicated Cloud (Premium, tuned) | 1000s+ | ~10–100ms (warm) | 0.9–0.99+ | Latency-sensitive search at scale |
| Self-hosted (optimized, BQ/RQ) | up to ~10,000 | <100ms (warm) | 0.96–0.99+ | Large-scale custom deployments |
| With external vectorizers | Variable | +50–200ms extra | Same | Online embedding generation |

These ranges align with Weaviate’s own ANN benchmarks (showing high QPS for tuned HNSW and compressed indexes) and third-party comparisons where Weaviate achieves several hundred to 1000+ QPS on realistic datasets when sized correctly.

## Pros

### ✅ Flexible Vectorization System

- Pluggable modules for popular embedding providers plus a managed embedding add-on.
- Option to either vectorize at ingestion/query time or use pre-computed embeddings.
- Model changes often require only config updates, not data migrations.

### ✅ Strong Hybrid Search

- Native BM25 + vector search with score fusion strategies.
- Works well for catalog, documentation, and help-center search.
- Supports multiple vector fields per object for different modalities or models.

### ✅ Open Source Foundation

- Core engine is open source and can be self-hosted.
- Active community, documentation, and ecosystem tooling.
- Self-hosted Weaviate uses the same engine as Weaviate Cloud.

### ✅ Developer-Friendly Features

- GraphQL, REST, and gRPC APIs.
- Integrations and examples for LangChain, LlamaIndex, and other frameworks.
- Best-practice guidance for indexing, multi-tenancy, and operations.

## Cons (Areas for Awareness & Planning)

### ⚙️ Performance Challenges at Scale

- For **multi-billion vectors** with tight (<50ms) global SLOs, you must carefully tune sharding, compression, and hardware; naive configs can see P99 latencies in the 200–500ms range under heavy load.
- Hybrid search (BM25 + vector) and complex filters are more expensive than pure vector ANN and require ACORN and filter tuning to maintain performance.

### ⚙️ Complex Horizontal Scaling

- Effective cluster design (shards, replicas, memory limits) still requires experience.
- **Multi-tenancy uses one shard per tenant**, which simplifies isolation but means a single “heavy” tenant is bound to one shard’s capacity; you must split that tenant at the application level if it grows too large.
- Multi-region or cross-cloud architectures typically need direct involvement from experienced operators or Weaviate’s team.

### ⚙️ Vectorizer Dependencies

- In-database vectorization depends on external LLM/embedding providers; latency and rate limits can affect ingestion and query-time performance.
- For strict latency and reliability, many teams pre-compute vectors and treat Weaviate purely as a vector+metadata store.

### ⚙️ Known Issues

Recent community discussions highlight:

- Operational complexity for self-hosted clusters (upgrades, scaling, observability).
- Challenges scaling very large tenants under the one-shard-per-tenant constraint.
- Lack of a full GUI admin console for self-hosted; monitoring relies on metrics/logs and external tooling.

## Benchmarks

Performance information comes from:

- Official docs and blogs (e.g., ACORN filtering and quantization benchmarks).
- Community comparisons (e.g., Weaviate vs Qdrant vs Pinecone).
- Marketplace and independent reviews that include basic QPS/tail latency numbers.

Key points:

- HNSW with uncompressed vectors can achieve **high QPS with sub-100ms** latency on properly sized hardware.
- Quantization (Binary and Rotational) trades a small recall loss for significant memory savings and often similar or better throughput.
- ACORN improves filtered query performance significantly on large datasets with low-correlation filters.

Actual production performance is highly workload-dependent; custom benchmarks on representative data are still required.

## When to Choose Weaviate

### ✅ Good Fit

- You need **hybrid search** (semantic + keyword) for catalogs, docs, or support portals.
- You value **open-source**, with the option to move between self-hosted and managed cloud.
- You’re building **multi-tenant SaaS** where tenant isolation, lazy loading, and lifecycle management matter.
- You want flexibility in embedding providers and may change models over time.
- You’re willing to invest some effort into schema and index tuning for performance/cost efficiency.

### ❌ Consider Alternatives

- You need ultra-low latency (<30ms P99) at **massive global scale** and prefer a more opinionated, specialized platform.
- You want a “zero-ops” experience with minimal tuning and configuration.
- Your team lacks capacity for basic infra/observability, and you don’t want managed cloud either.
- You only need simple vector-only search without hybrid/multi-tenancy features and want the simplest possible API.

## Alternatives to Consider

- **For peak performance / simplicity**: highly opinionated managed vector stores with serverless RU-style pricing.
- **For simplicity and limited features**: lighter-weight hosted vector search services.
- **For SQL-centric stacks**: PostgreSQL + pgvector, SingleStore, and similar.
- **For minimum infra cost**: self-hosted Weaviate, Qdrant, or OpenSearch if you already have strong DevOps capacity.

## Real-World Cost Examples

> All previous cost examples using $0.095 / $0.145 / $0.175 per 1M dims and AIU multipliers are stale. The following recomputes each scenario using the 2025 Flex/Plus/Premium model (vector dimensions + storage + backups), with approximate rates from updated docs.
> 

### Example 1: E-commerce Product Search

**Scenario**: Online retailer with 1M products, daily catalog updates

- **Vectors**: 1M product embeddings (768-dim) = **768M dimensions**, ~**2.9 GiB** storage.
- **Daily writes**: 50K product updates
- **Monthly reads**: 2M customer searches
- **Metadata**: Product attributes (category, price, etc.)

**Monthly Cost Breakdown (Shared Cloud):**

```
Flex:
- Base fee: ≈ $45
- Vector dimensions: 768M × $0.000327 ≈ $0.25
- Storage: 2.9 GiB × $0.2125 ≈ $0.62
- Backups: 2.9 GiB × $0.022 ≈ $0.06
Total: ~ $46/month

Plus:
- Base fee: ≈ $280
- Vector dimensions: same ≈ $0.25
- Storage: same ≈ $0.62
- Backups: same ≈ $0.06
Total: ~ $281/month

```

### Example 2: Large-Scale RAG Application

**Scenario**: Enterprise knowledge base with document chunks

- **Vectors**: 100M document chunks (1536-dim) = **153.6B dimensions** (153,600M).
- **Daily writes**: 1M new chunks
- **Monthly reads**: 50M user queries
- **Metadata**: Rich document metadata, timestamps, permissions
- **Storage**: ~**586 GiB** (vectors + indexes + metadata, assuming modest compression).

**Monthly Cost Breakdown (Plus, Shared or small Dedicated):**

```
Plus:
- Base fee: ≈ $280
- Vector dimensions: 153,600M × $0.000327 ≈ $50.23
- Storage: 586 GiB × $0.2125 ≈ $124.53
- Backups: 586 GiB × $0.022 ≈ $12.89
Total: ~ $468/month

```

> For HA with multiple replicas, both dimensions and storage scale roughly linearly with the number of replicas; this would increase the usage component accordingly, but not reintroduce the old AIU “3× multiplier” semantics.
> 

### Example 3: Real-time Recommendation Engine

**Scenario**: Streaming platform with user behavior vectors

- **Vectors**: 10M user profiles + 1M content items (512-dim)
- **Total**: 11M × 512 = **5.6B dimensions** (≈ 5,632M).
- **High write volume**: 5M daily interactions
- **Very high reads**: 100M monthly recommendations
- **Real-time requirements**: Sub-50ms latency (for warm data paths)
- **Storage**: ~**21 GiB** combined vectors + metadata.

**Monthly Cost Breakdown (Plus, Shared Cloud):**

```
Plus:
- Base fee: ≈ $280
- Vector dimensions: 5,632M × $0.000327 ≈ $1.84
- Storage: 21 GiB × $0.2125 ≈ $4.46
- Backups: 21 GiB × $0.022 ≈ $0.46
Total: ~ $287/month

```

For this workload, **Dedicated Cloud (Premium)** becomes attractive mainly for:

- Stricter global SLOs (e.g., P95 < 50ms across regions),
- Compliance and private networking,
- Or consolidating many similar workloads on dedicated infrastructure.

### Cost Tipping Points

| Workload Type | Serverless (Flex / Plus) Wins | Enterprise / Premium Wins |
| --- | --- | --- |
| **Prototyping/Dev** | <10M vectors, relaxed SLAs, low budgets | Rarely needed |
| **Production Search** | Up to ~100M vectors in one region | Very high QPS, strict SLAs, multi-region reads |
| **High-Performance RAG** | Medium corpora with compression + tuned ACORN | Massive corpora with complex filters + compliance |
| **Multi-tenant SaaS** | Hundreds–low thousands of tenants per cluster | Tens of thousands of high-traffic tenants |

**Critical Insight**: Under the 2025 model, **vector dimensions are cheap**; storage, backups, and plan minimums dominate. Compression (RQ/BQ) and tenant lifecycle management (offloading inactive tenants) are the main levers to control cost at scale.

## Bottom Line

Weaviate remains a strong choice for teams that:

- Need **hybrid search**, multi-tenancy, and flexible vectorization.
- Want the option to move between self-hosted OSS and fully managed cloud.
- Are willing to design schemas, indexes, and multi-tenant layouts with some care.

The **old** Weaviate pricing story—“$0.095 per 1M dims, AIUs for Enterprise, and HA as a simple 3× multiplier”—is now outdated. The **current** model is:

- **Conceptually simple** (vector dimensions + storage + backups),
- **Operationally realistic** (HA, backups, and retention are explicit),
- And **better aligned** with how engineers actually tune cost (compression, data tiering, tenant lifecycle).

**Recommendation**:

- Start on **Flex** or self-hosted OSS for experiments and smaller RAG/search workloads.
- Move to **Plus** once you have stable production traffic and need stronger SLAs.
- Consider **Premium Dedicated Cloud** when you require strict latency/compliance guarantees or are consolidating many production tenants into a single enterprise platform.

---

*Last updated: November 2025 | Reflects Weaviate’s October 2025 pricing update and current multi-tenancy and performance guidance.*