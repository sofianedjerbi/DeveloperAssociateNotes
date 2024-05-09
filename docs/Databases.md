# Databases

## Caching Strategies

Strategies **can be combined**.

### Lazy Loading / Cache Aside / Lazy Population

First, **ask the cache**, if there's a cache miss **ask the DB**.  

*Pros: Cache is not filled with unused data*  
*Cons: TTL + read penalty*

### Write Through

On every DB write, we also **write to the cache**.  

*Pros: Write penalty, cache is never stale*  
*Cons: Missing data until it is added / updated in the DB*

## Cache Evictions

**Cache Eviction** can occur in three ways:
- **Explicit** delete
- Item evicted because **memory is full**
- **TTL** (Time To Live)

## Amazon MemoryDB fpr Redis

MemoryDB is a **Redis-compatible** in-memory **database**.  
It offer **ultra-fast performance**. It scales from 10GBs to 100TBs
