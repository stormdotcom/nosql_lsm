 Database Components Documentation
 
1️⃣ Write-Ahead Log (WAL)
Purpose:

The Write-Ahead Log (WAL) ensures durability and data integrity by logging every write operation before applying it to the in-memory structure (MemTable). This protects data against system crashes.
How It Works:

    Every write (PUT or DELETE) is first appended to the WAL on disk.
    Once logged, the write is applied to the MemTable.
    In case of a system failure, the WAL is replayed to recover the data.

Advantages:

    Guarantees durability: Data is safe even if the system crashes.
    Provides fast recovery by replaying logs.

Example Workflow:

    Client sends PUT("key1", "value1").
    Append "key1:value1" to the WAL.
    Insert "key1":"value1" into the MemTable.
    On a crash, the WAL is read, and the MemTable is rebuilt.

2️⃣ MemTable (In-Memory Tree)
Purpose:

The MemTable is a fast, in-memory data structure (typically a balanced binary search tree like a Red-Black Tree or AVL Tree) that stores active data.
How It Works:

    New writes are added to the MemTable after being logged in the WAL.
    When the MemTable grows beyond a threshold, it is flushed to disk as an SSTable.
    The MemTable supports quick reads and writes.

Advantages:

    High write throughput due to in-memory operations.
    Supports efficient data retrieval using sorted structures.

Example Workflow:

    PUT("key2", "value2") → WAL → MemTable.
    MemTable reaches size limit → Flushed to SSTable.

3️⃣ SSTables (Sorted String Tables)
Purpose:

SSTables are immutable, sorted files stored on disk. They are created when the MemTable is flushed.
How It Works:

    MemTable data is written to disk in a sorted format.
    Each SSTable is immutable and optimized for sequential reads.
    An index is maintained for faster lookups.
    Over time, many SSTables are created and later merged by compaction.

Advantages:

    Efficient storage on disk due to sorting.
    Simplifies write operations by avoiding in-place updates.

Example Workflow:

    MemTable flushes sorted data → sstable_1.db.
    Newer writes go to MemTable → flushed into sstable_2.db.

4️⃣ Bloom Filter
Purpose:

A Bloom Filter is a space-efficient, probabilistic data structure used to quickly check if a key might exist in the database.
How It Works:

    Each key is hashed using multiple hash functions.
    Bits are set in a bit array based on these hashes.
    On a read, if any bit is unset, the key definitely does not exist.
    If the bits are set, the key might exist → check SSTables.

Advantages:

    Reduces disk I/O by filtering out non-existent keys.
    Highly space-efficient.

Limitations:

    False positives are possible (key might exist when it doesn’t).
    False negatives are impossible.

Example Workflow:

    Insert "key1" → hashes → bits set.
    GET("key2") → Bloom filter returns false → skip SSTable lookup.
    GET("key1") → Bloom filter returns true → search SSTable.

5️⃣ Compaction
Purpose:

Compaction is a background process that merges multiple SSTables, removes duplicate entries, and reclaims space.
How It Works:

    Multiple SSTables are merged into a new, sorted SSTable.
    Duplicate keys are replaced by the most recent value.
    Deleted keys (tombstones) are permanently removed.
    Older SSTables are deleted after merging.

Advantages:

    Reduces read amplification by minimizing the number of SSTables.
    Frees up disk space by removing redundant data.
    Keeps data optimized for faster reads.

Types of Compaction:

    Minor Compaction: Merges small SSTables.
    Major Compaction: Merges all SSTables into one.

Example Workflow:

    sstable_1.db → {key1:value1, key2:value2}
    sstable_2.db → {key2:value3, key3:value4}
    Compaction merges → {key1:value1, key2:value3, key3:value4}
    Deletes sstable_1.db and sstable_2.db.

🔄 Overall Data Flow

    Write (PUT):
        Log to WAL → Write to MemTable → Bloom filter updated.
        MemTable flushes to SSTable when full.

    Read (GET):
        Check Bloom Filter → MemTable → SSTables (newest to oldest).

    Delete (DELETE):
        Insert a tombstone marker in the MemTable.
        Tombstones are cleaned during Compaction.

    Recovery:
        Replay WAL to restore MemTable after a crash.

📊 Advantages of LSM Tree-based Databases
✅ High Write Throughput

    Sequential writes in WAL and MemTable are fast.

✅ Efficient Reads

    Bloom filters and sorted SSTables allow quick lookups.

✅ Space Efficiency

    Compaction removes redundant data and optimizes storage.

✅ Fault Tolerance

    WAL ensures durability and crash recovery.

📝 Conclusion

The LSM Tree architecture with its core components—WAL, MemTable, SSTables, Bloom Filter, and Compaction—provides a scalable and efficient way to handle large-scale data in NoSQL databases. This structure balances the needs for high write throughput, efficient reads, and data durability.
