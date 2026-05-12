
# Part 2: Pull Request Analysis

Repository: https://github.com/aio-libs/aiokafka

Selected Pull Requests:
1. PR #115 – Fix for compacted topic offsets
2. PR #143 – Metadata listener support when `group_id=None`

---

# PR 1 — PR #115

## PR Link
https://github.com/aio-libs/aiokafka/pull/115

## PR Title
Added fix to support compacted topics data, where some offsets can be missing

---

## PR Summary

This PR fixes an issue related to Kafka compacted topics. In Kafka, compacted topics may remove older messages to save storage space. Because of this, some offsets may no longer exist even though newer messages are still available.

The problem happened when the aiokafka consumer tried to read messages sequentially. The consumer internally expected offsets to exist continuously. When Kafka removed some records during topic compaction, gaps appeared between offsets. This caused the consumer fetch logic to behave incorrectly and sometimes repeatedly fetch the same batch again.

This PR improves how aiokafka handles missing offsets inside compacted topics. Instead of assuming all offsets exist continuously, the consumer now safely skips removed offsets and correctly moves to the next valid fetch position.

This makes Kafka consumers more reliable when working with compacted topics.

---

## Technical Changes

Files modified:

- `aiokafka/fetcher.py`
- `aiokafka/producer.py`

Main components affected:

- Fetcher logic
- Offset tracking system
- Batch processing logic
- Consumer fetch handling
- Compacted topic support

---

## Implementation Approach

This PR mainly updates the Kafka fetcher logic inside `fetcher.py`.

The old implementation expected offsets to exist continuously. For example:

```text
1 → 2 → 3 → 4
```

But compacted topics can contain gaps like:

```text
1 → 3 → 5
```

because Kafka removes older duplicate records during compaction.

The old fetch logic could become confused when offsets were missing. In some situations, the consumer repeatedly fetched the same batch because the internal fetch offset was not updated correctly.

The fix changes how `next_fetch_offset` is updated after processing batches. Instead of assuming the next offset should always increase sequentially, the consumer now uses Kafka batch metadata to move directly to the next valid offset.

Additional logic was also added to safely skip already removed or outdated records.

In simple terms, the fetcher became more flexible and now properly handles missing offsets without getting stuck or repeatedly fetching the same messages.

---

## Potential Impact

This PR mainly affects Kafka consumers using compacted topics.

Systems that use:
- stream processing
- event sourcing
- state synchronization
- distributed caching
- log compaction

benefit from this improvement.

The changes improve:
- consumer stability
- fetch reliability
- offset management
- compacted topic compatibility

Without this fix, some consumers could repeatedly fetch the same batches or fail to progress correctly.

---

# PR 2 — PR #143

## PR Link
https://github.com/aio-libs/aiokafka/pull/143

## PR Title
Added metadata change listener if group_id is None

---

## PR Summary

This PR fixes an issue where Kafka consumers using `group_id=None` could not automatically detect newly created topics when subscribing using topic patterns.

Normally, Kafka consumer groups automatically handle metadata updates and partition assignments. But when `group_id=None` is used, the consumer works independently without Kafka group coordination.

The problem was that standalone consumers were not listening for Kafka metadata updates correctly. Because of this, if a new topic matching the subscription pattern was created later, the consumer would never discover it automatically.

This PR adds metadata change listeners for standalone consumers using `NoGroupCoordinator`.

After this fix, consumers can automatically refresh metadata, detect newly created matching topics, and assign their partitions dynamically.

This improves dynamic topic discovery for standalone consumers.

---

## Technical Changes

Files modified:

- `aiokafka/consumer.py`
- `aiokafka/group_coordinator.py`
- `aiokafka/fetcher.py`

Main components affected:

- Metadata update listeners
- Pattern subscription handling
- NoGroupCoordinator logic
- Partition assignment system
- Consumer topic discovery

---

## Implementation Approach

This PR mainly improves metadata handling for consumers running without a consumer group.

When a consumer uses `group_id=None`, aiokafka creates a `NoGroupCoordinator` instead of the normal Kafka group coordinator. Previously, this standalone coordinator did not properly listen for Kafka metadata changes.

The fix adds metadata listeners that react whenever Kafka topic metadata changes. When new topics are created, the consumer now refreshes metadata and checks whether the new topics match the subscribed pattern.

For example, if a consumer subscribes using:

```python
pattern="logs-*"
```

and a new topic like `logs-payment` is later created, the consumer can now automatically discover it.

The PR also updates partition assignment handling so newly matched topics are assigned correctly without restarting the consumer.

In simple terms, this PR allows standalone consumers to dynamically detect and consume newly created matching topics.

---

## Potential Impact

This PR mainly affects consumers using:
- pattern-based subscriptions
- standalone consumers
- dynamic topic discovery

The changes improve:
- topic discovery
- metadata refresh handling
- partition assignments
- standalone consumer behavior

Without this fix, standalone consumers could miss newly created topics unless they were manually restarted.

---

# Integrity Declaration

"I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words."

