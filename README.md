# Database Components Documentation

## 1️⃣ Write-Ahead Log (WAL)

### Purpose:
The Write-Ahead Log (WAL) ensures durability and data integrity by logging every write operation before applying it to the in-memory structure (MemTable). This protects data against system crashes.

### How It Works:
1. Every write (PUT or DELETE) is first appended to the WAL on disk.
2. Once logged, the write is applied to the MemTable.
3. In case of a system failure, the WAL is replayed to recover the data.

### Advantages:
- **Guarantees durability:** Data is safe even if the system crashes.
- **Provides fast recovery:** Logs are replayed to recover data.

### Example Workflow:
1. Client sends `PUT("key1", "value1")`.
2. Append `"key1:value1"` to the WAL.
3. Insert `"key1":"value1"` into the MemTable.
4. On a crash, the WAL is read, and the MemTable is rebuilt.

---

## 2️⃣ MemTable (In-Memory Tree)

### Purpose:
The MemTable is a fast, in-memory data structure (e.g., Red-Black Tree, AVL Tree) that stores active data.

### How It Works:
1. New writes are added to the MemTable after being logged in the WAL.
2. When the MemTable grows beyond a threshold, it is flushed to disk as an SSTable.
3. The MemTable supports quick reads and writes.

### Advantages:
- **High write throughput:** Operations are in-memory.
- **Efficient data retrieval:** Sorted structures optimize lookups.

### Example Workflow:
1. `PUT("key2", "value2")` → WAL → MemTable.
2. MemTable reaches size limit → Flushed to SSTable.

---

## 3️⃣ SSTables (Sorted String Tables)

### Purpose:
SSTables are immutable, sorted files stored on disk, created when the MemTable is flushed.

### How It Works:
1. MemTable data is written to disk in a sorted format.
2. Each SSTable is immutable and optimized for sequential reads.
3. An index is maintained for faster lookups.
4. Over time, many SSTables are created and later merged by compaction.

### Advantages:
- **Efficient storage:** Data is stored in a sorted format.
- **Simplifies writes:** No in-place updates required.

### Example Workflow:
1. MemTable flushes sorted data → `sstable_1.db`.
2. Newer writes go to MemTable → Flushed into `sstable_2.db`.

---

## 4️⃣ Bloom Filter

### Purpose:
A Bloom Filter is a space-efficient, probabilistic data structure used to quickly check if a key might exist in the database.

### How It Works:
1. Each key is hashed using multiple hash functions.
2. Bits are set in a bit array based on these hashes.
3. On a read:
   - If any bit is unset, the key definitely does not exist.
   - If the bits are set, the key might exist → Check SSTables.

### Advantages:
- **Reduces disk I/O:** Filters out non-existent keys.
- **Space-efficient.**

### Limitations:
- False positives are possible (key might exist when it doesn’t).
- False negatives are impossible.

### Example Workflow:
1. Insert `"key1"` → Hashes → Bits set.
2. `GET("key2")` → Bloom filter returns false → Skip SSTable lookup.
3. `GET("key1")` → Bloom filter returns true → Search SSTable.

---

## 5️⃣ Compaction

### Purpose:
Compaction is a background process that merges multiple SSTables, removes duplicate entries, and reclaims space.

### How It Works:
1. Multiple SSTables are merged into a new, sorted SSTable.
2. Duplicate keys are replaced by the most recent value.
3. Deleted keys (tombstones) are permanently removed.
4. Older SSTables are deleted after merging.

### Advantages:
- **Reduces read amplification:** Minimizes the number of SSTables.
- **Frees up disk space:** Removes redundant data.
- **Optimizes reads:** Keeps data sorted.

### Types of Compaction:
- **Minor Compaction:** Merges small SSTables.
- **Major Compaction:** Merges all SSTables into one.

### Example Workflow:
1. `sstable_1.db` → `{key1:value1, key2:value2}`
2. `sstable_2.db` → `{key2:value3, key3:value4}`
3. Compaction merges → `{key1:value1, key2:value3, key3:value4}`
4. Deletes `sstable_1.db` and `sstable_2.db`.

---

## 🔄 Overall Data Flow

### Write (PUT):
1. Log to WAL → Write to MemTable → Bloom filter updated.
2. MemTable flushes to SSTable when full.

### Read (GET):
1. Check Bloom Filter → MemTable → SSTables (newest to oldest).

### Delete (DELETE):
1. Insert a tombstone marker in the MemTable.
2. Tombstones are cleaned during Compaction.

### Recovery:
- Replay WAL to restore MemTable after a crash.

---

## 📊 Advantages of LSM Tree-based Databases

### ✅ High Write Throughput:
- Sequential writes in WAL and MemTable are fast.

### ✅ Efficient Reads:
- Bloom filters and sorted SSTables allow quick lookups.

### ✅ Space Efficiency:
- Compaction removes redundant data and optimizes storage.

### ✅ Fault Tolerance:
- WAL ensures durability and crash recovery.

---

## 📝 Conclusion
The LSM Tree architecture with its core components—WAL, MemTable, SSTables, Bloom Filter, and Compaction—provides a scalable and efficient way to handle large-scale data in NoSQL databases. This structure balances the needs for high write throughput, efficient reads, and data durability.
