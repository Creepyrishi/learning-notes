**1. What FAISS is**
FAISS (Facebook AI Similarity Search) is a library for fast similarity search over vector embeddings. It retrieves nearest vectors to a query vector using different indexing strategies.


**2. Core Idea**
Workflow:
Text → Embedding → Vector Index → Similarity Search → Top-K Results → Map to original text (RAG retrieval)


**3. Indexes (Main Search Engine Types)**

**A. Flat Indexes (Exact Search)**
- IndexFlatIP → Inner Product (cosine similarity when normalized)
- IndexFlatL2 → Euclidean distance
- Behavior: brute-force scan of all vectors
- Pros: exact results
- Cons: slow at scale (O(N))


**B. IVF Indexes (Cluster-Based Approximate Search)**
- IndexIVFFlat → clusters + exact search inside selected clusters
- IndexIVFPQ → IVF + compression (Product Quantization)
- IndexIVFSQ → scalar quantization variant

Mechanism:
1. Cluster vectors (k-means)
2. Assign vectors to centroids
3. Query searches only nearest clusters (controlled by nprobe)

Tradeoff: speed vs accuracy


**C. HNSW Index (Graph-Based Search)**
- IndexHNSWFlat
Mechanism:
- Builds graph of vector neighbors
- Search by traversing closest nodes

Pros:
- Very fast
- High recall
Cons:
- Memory heavy


**D. Compression Indexes**
- PQ (Product Quantization)
- IVFPQ (IVF + PQ compression)

Purpose:
- Reduce memory footprint
- Store compressed vector codes instead of full floats


**4. ID Wrappers (Output Mapping Layer)**

**IndexIDMap**
Maps FAISS internal positions → real-world IDs (chunk_id, doc_id)
Without:
- returns 0,1,2,...

With:
- returns meaningful IDs for RAG retrieval

**IndexIDMap2**
Advanced version with better flexibility for dynamic updates

Key point:
- Does NOT affect search, only output identity


**5. Training Component**

Required for:
- IVF (clustering)
- PQ (codebook learning)

Function:
- index.train(data)

What it does:
- Learns clusters (IVF)
- Learns quantization codebooks (PQ)


**6. Vector Transformations (Preprocessing)**

- Normalization → unit length vectors (enables cosine via inner product)
- PCA → dimensionality reduction
- OPQ → optimized rotation for better compression


**7. Similarity Metrics**

- Inner Product (IP): used for cosine similarity when vectors are normalized
- L2 Distance: Euclidean distance


**8. Search Optimization Parameters**

- IVF: nprobe → number of clusters searched
- HNSW: efSearch → graph traversal depth

Higher = more accurate, slower


**9. GPU Acceleration**

- GPU IndexFlatIP / GPU IVF
- index_cpu_to_gpu()

Used for:
- large-scale / high-throughput inference


**10. Index Factory**

faiss.index_factory()

Examples:
- "Flat"
- "IVF100,Flat"
- "IVF100,PQ16"
- "HNSW32"

Purpose:
- declarative index creation


**11. Persistence Layer**

- write_index()
- read_index()

Used to save/load FAISS indexes in production


**12. Metadata Layer (External System)**

FAISS does NOT store text.

You must maintain:
- chunk text
- document IDs
- metadata

IndexIDMap connects vectors → metadata system


**13. Full RAG Flow**

1. Chunk text
2. Embed (BGE-m3 or similar)
3. Normalize vectors
4. Build FAISS index
   - Flat / IVF / HNSW
   - optionally IDMap
5. Query embedding
6. FAISS search
7. Retrieve chunk IDs
8. Fetch actual text from DB


**14. Big Picture Mental Model**

- Flat → brute force exact search
- IVF → clustered approximate search
- HNSW → graph traversal search
- PQ → compressed storage
- IDMap → identity mapping layer
- Training → structure learning step
- GPU → acceleration layer


**15. Key Engineering Tradeoffs**

- Flat: accuracy ↑, speed ↓
- IVF: balanced
- HNSW: fast + accurate, memory heavy
- PQ: memory efficient, less accurate


In work

3 main problems in FAISS:
1. while building i am fetching all docs and then build. so if a new doc is just added whole entire building will happen
2. as loading is cpu bound task it am blocking the thread for long, it would be better if i just move the building and load in other thread or proceess  or celery

**By claude**
**Recommendation: Thread.** Two reasons, both decisive here:
- **FAISS releases the GIL** during its heavy C++ ops (`read_index`, `write_index`,  
    `add_with_ids`, `search`). Threads only fail to give real parallelism for _pure-Python_  
    CPU work; they work great for GIL-releasing native libraries like FAISS and numpy. So a thread  keeps the event loop responsive while the C call runs.
- **Threads share memory.** The index built in a thread lives in the same process, so it can be  
    stored in the module global and read by the query path directly — which is exactly what "keep it  in RAM" needs. A **Process** builds the index in a _separate_ address space; to keep it in RAM you'd  have to pickle the whole FAISS index back across the process boundary (expensive IPC) — defeating  the purpose. Processes also cost far more to spawn.

	For the same reason we wont celery.

3.  i was loading the index and then remove the index for every query i was doing which very bad. here i have multiple thing that i can do. 1. move the loading to diffrent thread or process 2. i cannot use the use celery as celery some problem