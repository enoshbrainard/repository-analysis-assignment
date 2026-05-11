# Part 4: Technical Communication

I selected PR #115 because it was easier for me to understand compared to many of the other pull requests. Some PRs involved larger architectural changes or deeper Kafka coordination logic, but this PR focused mainly on Kafka consumer offset handling for compacted topics.

I already have some backend development knowledge from working with asynchronous systems in Node.js. Even though this repository is written in Python, I was still able to understand the async flow, message fetching, and consumer behavior. The producer-consumer pattern used in Kafka was also easier for me to relate to because I already understood how backend services communicate using events and messages.

What made this PR more understandable was that the problem was very specific. Kafka compacted topics can remove older messages, which creates gaps between offsets. The older fetch logic expected offsets to increase continuously, which caused problems while fetching batches. After reading the changes inside `fetcher.py`, I understood that the fix mainly focused on correctly updating the next fetch offset instead of assuming every offset exists.

One challenge I would expect while implementing this type of PR is understanding all Kafka edge cases related to compacted batches and compressed records. Since the fetcher handles many records internally, small mistakes in offset tracking could create repeated fetch loops or skipped messages.

If I were implementing this myself, I would try to overcome these challenges by carefully tracing the fetch flow step by step, reading Kafka batch metadata behavior, and testing different compacted topic scenarios. I would also add test cases for missing offsets and verify that the consumer continues progressing correctly without repeatedly fetching the same batch.

---

# Integrity Declaration

"I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words."
