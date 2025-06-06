# SingleStore Review

## Overview

SingleStore is a distributed, relational SQL database management system that has integrated robust vector search capabilities. Unlike pure vector databases, SingleStore is a general-purpose, high-performance operational database that supports vector data as a native type with specialized indexing and functions for similarity search. Its architectural approach provides a unified platform for transactional, analytical, and vector workloads, aiming to reduce data silos and simplify data architectures.

## Architecture

### Core Architecture
- **Distributed SQL Database**: Built as a distributed system with horizontal scaling capabilities through sharding and replication
- **Unified Platform**: Combines OLTP (transactional) and OLAP (analytical) processing with vector search in a single system
- **Columnstore Storage**: Vector data typically stored in columnstore tables for optimal performance
- **Native Vector Type**: VECTOR(<N>, F32) data type with dimensions up to 16,000
- **Pluggable Index Architecture**: Allows integration of new vector index libraries (Faiss, Knowhere, etc.)

### Key Features
- SQL-based vector operations with full ACID compliance
- Real-time analytics with sub-second query response
- Hybrid search combining vector similarity with SQL predicates
- Change Data Capture (CDC) for real-time data synchronization
- High availability with disaster recovery capabilities

## Pricing Models

### Primary Pricing Model
**Credit-Based Pricing (SingleStore Helios Cloud):**
- Workspace Units: Starting at $0.90/hr (Standard) or $1.35/hr (Enterprise)
- Storage: $0.023-$0.025 per GB/month
- Data Ingress: Free (no charge for data transfer in)
- Credits: $3.6 per credit

**Example Cost (10M vectors, 768-dim):**
- ~$144/month for S00 workspace (dev/test usage)
- ~$31,824/month for production S4+S8 workspaces

### Alternative Pricing Model
**Resource-Based Tiers:**
- Free Tier: Shared workspace for evaluation
- Standard: Starting at $0.90/hr with 99.9% SLA
- Enterprise: Starting at $1.35/hr with 99.99% SLA (multi-AZ)

**Example Cost (same workload):**
- Free tier available for small workloads
- Production costs scale with compute resources (S00 to S8+ workspaces)

## Performance Characteristics

| Configuration | QPS | P99 Latency | Recall | Use Case |
|---------------|-----|-------------|--------|----------|
| S-2 Cluster (HNSW) | 2nd best* | Decreases with scale | 88.8-91.5% | High-throughput search |
| S-4 Cluster (HNSW) | Higher | Lower | Similar | Production workloads |
| IVF_PQFS Index | Good | Moderate | ~85-90% | Memory-constrained |
| HNSW_FLAT Index | Excellent | Low | >95% | High-accuracy needs |

*benchANT benchmark comparing with Pinecone and Zilliz

## Pros

### ✅ Unified Data Platform
- Single system for transactional, analytical, and vector workloads
- Reduces architectural complexity and data movement
- Leverages existing SQL expertise and tools
- Enables powerful queries combining vectors with joins, aggregations, and filters

### ✅ Performance & Scalability
- 800-1,000x faster than exact kNN methods with ANN indexes
- Comparable performance to specialized vector databases (Milvus, Pinecone)
- Significantly outperforms pgvector
- Horizontal scaling with distributed architecture

### ✅ Enterprise Features
- ACID transactions for data consistency
- High availability with automatic failover
- Point-in-time recovery and disaster recovery
- Comprehensive security features (encryption, RBAC, audit logging)

### ✅ Developer Experience
- Standard SQL interface with vector extensions
- Multiple index types (AUTO, HNSW, IVF variants) with automatic selection
- Native support for JSON, BSON, and vector data types
- Integration with popular ML frameworks

## Cons

### ❌ Vector-Specific Limitations
- Currently only supports F32 element type for vectors
- May not match absolute peak performance of top-tier specialized vector databases
- Less granular control over vector-specific tuning compared to dedicated solutions
- Vector index libraries still being integrated and optimized

### ❌ Complexity for Simple Use Cases
- Overkill for applications that only need vector search
- Requires understanding of distributed SQL systems for optimal configuration
- Higher operational overhead than simple vector-only databases
- Resource contention possible between vector and other workloads

### ❌ Cost Considerations
- Can be expensive at scale compared to open-source alternatives
- Memory requirements for high-performance vector indexes (HNSW)
- Enterprise features only available in higher tiers
- Compute costs scale with workspace size

### ❌ Known Issues
Recent developments and limitations:
- Vector search is a relatively new addition (indexed ANN released mid-2024)
- Documentation for vector-specific optimizations still evolving
- Limited community resources compared to established vector databases
- Potential learning curve for teams new to distributed SQL

## Benchmarks

Performance data from various sources:
- **benchANT Study (2025)**: Second-best QPS after Zilliz, outperformed Pinecone
- **SingleStore Internal**: 800-1,000x faster than exact kNN, comparable to Milvus
- **vs pgvector**: Orders of magnitude faster performance

**⚠️ Note**: Performance highly dependent on index type, data characteristics, and query patterns. Custom benchmarking recommended.

## When to Choose SingleStore

### ✅ Good Fit
- Existing SQL-based infrastructure needing vector capabilities
- Applications requiring both structured data and vector search
- Need for complex queries combining vectors with filters, joins, aggregations
- Enterprise requirements for HA, DR, and ACID compliance
- Teams with strong SQL expertise

### ❌ Consider Alternatives
- Pure vector search workloads with no SQL requirements
- Budget-constrained projects (open-source vector DBs may be cheaper)
- Applications requiring cutting-edge vector-specific features
- Small-scale projects where unified platform benefits aren't needed
- Teams seeking maximum control over vector indexing algorithms

## Alternatives to Consider

- **Pure Vector Databases**: Pinecone, Qdrant, Weaviate, Milvus
- **SQL + Vector**: PostgreSQL with pgvector, OpenSearch, Elasticsearch
- **Managed Services**: Pinecone Serverless, Zilliz Cloud, Qdrant Cloud

## Real-World Cost Examples

### Example 1: E-commerce Product Search
**Scenario**: Online retailer with 1M products, daily catalog updates
- **Vectors**: 1M product embeddings (768-dim) = ~2.9 GB storage
- **Daily writes**: 50K product updates
- **Monthly reads**: 2M customer searches
- **Metadata**: Product attributes, prices, inventory

**Monthly Cost Breakdown:**
```
SingleStore Helios:
- S00 Workspace (dev): 160 hours = $144
- Storage: 3 GB × $0.023 = $0.07
- Data ingress: Free
Total: ~$144/month (dev/test)

Production (S4):
- Workspace: 720 hours × 4 credits = $10,368
- Storage: 3 GB × $0.023 = $0.07
Total: ~$10,368/month
```

### Example 2: Large-Scale RAG Application
**Scenario**: Enterprise knowledge base with document chunks
- **Vectors**: 100M document chunks (1536-dim) = ~586 GB storage
- **Daily writes**: 1M new chunks
- **Monthly reads**: 50M user queries
- **Requirements**: Sub-100ms latency, high availability

**Monthly Cost Breakdown:**
```
SingleStore Enterprise:
- S8 Workspace: 720 hours × 8 credits = $20,736
- Storage: 586 GB × $0.025 = $14.65
- HA/DR features included
Total: ~$20,751/month
```

### Example 3: Real-time Recommendation Engine
**Scenario**: Streaming platform with user behavior vectors
- **Vectors**: 10M users + 1M content (512-dim) = ~21 GB
- **High write volume**: 5M daily interactions
- **Very high reads**: 100M monthly recommendations
- **Real-time requirements**: Sub-50ms latency

**Monthly Cost Breakdown:**
```
SingleStore Standard:
- S4 Workspace: 720 hours × 4 credits = $10,368
- Storage: 21 GB × $0.023 = $0.48
Total: ~$10,368/month
```

### Cost Tipping Points

| Workload Type | SingleStore Wins | Specialized Vector DB Wins |
|---------------|------------------|---------------------------|
| **Mixed SQL + Vector** | Always (unified platform) | Never (requires two systems) |
| **Pure Vector Search** | >50M vectors with filters | <50M vectors, simple queries |
| **High Write Volume** | When needing transactions | Streaming updates only |
| **Complex Analytics** | Always (SQL capabilities) | Simple similarity only |

**⚠️ Critical Insight**: SingleStore becomes cost-effective when you need both SQL and vector capabilities, eliminating the need for multiple systems.

## Bottom Line

SingleStore excels for organizations needing a unified data platform that combines traditional SQL capabilities with modern vector search. The credit-based pricing offers flexibility for development and production workloads, but **costs can escalate significantly** at scale:

- **Sweet spot**: Mixed workloads requiring SQL + vectors, 10M-1B vectors
- **Danger zone**: Pure vector workloads >1B vectors where specialized DBs may be more cost-effective
- **Break-even**: When eliminating 2+ separate systems (OLTP + OLAP + Vector)

For production deployments, carefully evaluate:

1. **Total platform cost** vs. maintaining separate systems
2. **Performance requirements** for both SQL and vector operations
3. **Operational complexity** of unified vs. specialized solutions
4. **Enterprise features** needs (HA, DR, security, compliance)

**Recommendation**: Choose SingleStore when you need a "database of databases" - combining transactional, analytical, and vector workloads. **Avoid it** for pure vector search applications where specialized solutions offer better price/performance.

---

*Last updated: June 2025 | Based on benchANT studies, SingleStore documentation, and public benchmarks*