# Part 3: Prompt Preparation

Selected Pull Request: PR #115  
Repository: https://github.com/aio-libs/aiokafka  
PR Link: https://github.com/aio-libs/aiokafka/pull/115

---

# 3.1.1 Repository Context

aiokafka is a Python library used to work with Apache Kafka using async programming. It helps Python applications send and receive Kafka messages without blocking the program execution. The library is built using Python asyncio.

This repository is mostly used by backend developers who build distributed systems, microservices, and real-time applications. Instead of directly connecting services together, applications can exchange events and messages through Kafka.

The repository contains:
- Kafka consumers
- Kafka producers
- fetch handling logic
- offset tracking
- partition assignment
- group coordination

The main purpose of this project is to make Kafka communication easier in Python async applications.

For example, one service may produce an event like:

```text
Order Created
```

and another service may consume it for:
- notifications
- analytics
- payments
- delivery systems

aiokafka handles all communication between the application and Kafka brokers.

The repository mainly solves problems related to:
- async message processing
- event streaming
- distributed communication
- real-time systems

It is commonly used in backend systems where many services exchange events continuously.

---

# 3.1.2 Pull Request Description

This PR fixes a problem related to Kafka compacted topics.

In Kafka compacted topics, old messages can be removed to save storage space. Because of this, some offsets may disappear.

Example:

```text
1 → 2 → 3 → 4
```

after compaction may become:

```text
1 → 3 → 5
```

The old aiokafka fetch logic expected offsets to exist continuously. When offsets were missing, the consumer could behave incorrectly.

Sometimes the consumer repeatedly fetched the same batch because the internal fetch offset was not updated correctly.

This PR changes the fetch handling logic inside `fetcher.py`.

Instead of assuming offsets always increase one by one, the consumer now uses Kafka batch metadata to move to the next valid offset.

The new implementation safely skips missing offsets and continues processing valid records normally.

This prevents consumers from getting stuck while reading compacted Kafka topics.

The PR also updates some producer documentation related to compacted topics.

---

# 3.1.3 Acceptance Criteria

✓ When compacted topics contain missing offsets, the consumer should continue reading messages normally.

✓ The fetcher should correctly move to the next valid offset after processing a batch.

✓ The consumer should not repeatedly fetch the same batch again.

✓ Missing offsets should be skipped safely without crashing the consumer.

✓ Valid records should still be returned correctly.

✓ Existing behavior for normal topics should remain unchanged.

✓ Compressed batches should still work correctly after the fix.

---

# 3.1.4 Edge Cases

## Edge Case 1

The implementation should handle multiple missing offsets.

Example:

```text
1 → 5 → 10
```

The consumer should still continue correctly.

---

## Edge Case 2

Some compacted batches may contain no valid records.

The consumer should still update offsets correctly and continue fetching newer batches.

---

## Edge Case 3

Compressed Kafka batches may contain older offsets.

The implementation should safely skip old records without breaking fetch logic.

---

# 3.1.5 Initial Prompt

You are working on the `aiokafka` repository, which is an async Kafka client library for Python applications.

The task is to fix Kafka consumer handling for compacted topics where some offsets may be missing after Kafka compaction.

Currently, the fetcher logic assumes offsets always exist continuously. This creates problems when Kafka removes old records during compaction. In some situations, the consumer repeatedly fetches the same batch because the internal fetch offset is not updated correctly.

Update the fetch handling logic inside `fetcher.py` so the consumer can properly handle missing offsets.

The implementation should:
- safely skip missing offsets
- continue processing valid records
- correctly update `next_fetch_offset`
- avoid repeatedly fetching the same batch
- continue working for normal non-compacted topics

The implementation should use Kafka batch metadata to move to the next valid fetch position instead of assuming offsets always increase sequentially.

The solution should also handle:
- compacted topics with missing offsets
- compressed batches
- empty compacted batches

Testing requirements:
- test compacted topics with missing offsets
- test batches with offset gaps
- test empty compacted batches
- verify normal topics still work correctly
- ensure no infinite re-fetching happens

Focus mainly on the fetcher logic and offset handling implementation.

---

# Integrity Declaration

"I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words."
