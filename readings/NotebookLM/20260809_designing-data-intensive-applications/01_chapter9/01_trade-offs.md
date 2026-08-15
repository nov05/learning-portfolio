# 🟢 The 5 most important engineering trade-offs

<br><br><br>

## 👉 **Prompt generate by ChatGPT**

```text
Analyze Chapter 9 and identify the 5 most important engineering trade-offs.

For each trade-off use this structure:

Problem
→ Why the naive solution fails
→ Possible approaches
→ What each approach optimizes for
→ What each approach sacrifices
→ Failure modes
→ When to use it
→ When NOT to use it

Focus especially on:

- timeouts and failure detection
- retries and duplicate requests
- network reliability
- clocks and timestamps
- detecting whether a node is alive
- safety vs availability

Use concrete examples from the chapter and provide source citations.

Do not merely summarize the chapter. Explain the engineering decisions behind these trade-offs.
```

<br><br><br>

## 👉 **NoteboomLM output**  

Based on an analysis of Chapter 9, here are the five most critical engineering trade-offs for designing distributed systems. These decisions represent the shift from the "idealized" single-machine model to the messy physical reality of partial failures.

### 1. Failure Detection: Short vs. Long Timeouts
**Problem:** How to detect if a remote node is dead in an asynchronous network.
**→ Why the naive solution fails:** You cannot instantly know if a node has failed. In a packet-switched network, a request or its response could be lost, delayed by congestion, or the node might just be slow.
**→ Possible approaches:** 
1. **Short Timeouts:** Declare a node dead quickly (e.g., < 1 second).
2. **Long Timeouts:** Wait longer (e.g., 30+ seconds) before giving up.
**→ What each approach optimizes for:**
*   **Short:** Optimizes for **fast recovery**; a new leader can be elected immediately to restore service.
*   **Long:** Optimizes for **stability**; minimizes the risk of false positives where a healthy but slow node is incorrectly removed.
**→ What each approach sacrifices:**
*   **Short:** Sacrifices **reliability**; increases the risk of unnecessary failovers and "cascading failures".
*   **Long:** Sacrifices **availability**; users must wait through the entire timeout period during a real outage.
**→ Failure modes:** Short timeouts often trigger **retry storms** or cause a system to thrash by constantly electing new leaders when the network is just busy.
**→ When to use it:** Use short timeouts only if you have robust **fencing** to stop "zombie" nodes from doing damage.
**→ When NOT to use it:** Do not use short timeouts in public clouds where "noisy neighbors" cause unpredictable network jitter.

---

### 2. Network Design: Predictability vs. Resource Utilization
**Problem:** Designing the underlying network for either guaranteed performance or cost-efficiency.
**→ Why the naive solution fails:** Treating the network as a reliable utility (like the old telephone system) requires "circuit switching," which is prohibitively expensive for data.
**→ Possible approaches:**
1. **Circuit Switching:** Dedicating a fixed path and bandwidth to every connection.
2. **Packet Switching (TCP/IP):** Sharing bandwidth dynamically between all senders.
**→ What each approach optimizes for:**
*   **Circuit:** Optimizes for **predictability**; provides a "bounded delay" with zero queueing.
*   **Packet:** Optimizes for **utilization** and cost; maximizes the amount of data sent over the expensive physical wire.
**→ What each approach sacrifices:**
*   **Circuit:** Sacrifices **efficiency**; unused bandwidth is wasted, and the network can't handle "bursty" traffic well.
*   **Packet:** Sacrifices **timing guarantees**; introduces variable queueing delays and unbounded latency.
**→ Failure modes:** Packet switching leads to **congestive collapse** where several nodes saturate a switch, causing packets to be dropped even if the nodes are healthy.
**→ When to use it:** Use packet switching for almost all backend web services due to its cost-effectiveness.
**→ When NOT to use it:** Avoid packet switching for safety-critical "hard real-time" systems (e.g., aircraft controls) where a delay of 100ms could be fatal.

---

### 3. Event Ordering: Physical vs. Logical Clocks
**Problem:** Determining which of two events happened first across different machines.
**→ Why the naive solution fails:** Physical time-of-day clocks drift at different rates (quartz drift) and can be reset backward by NTP, making timestamps on different machines incomparable.
**→ Possible approaches:**
1. **Physical Clocks:** Use the machine's system time (Wall-clock).
2. **Logical Clocks:** Use incrementing counters (e.g., Lamport timestamps) to track causality.
**→ What each approach optimizes for:**
*   **Physical:** Optimizes for **human context** and absolute scheduling (e.g., "log out at 5:00 PM").
*   **Logical:** Optimizes for **causality**; ensures if A triggered B, A always has a lower timestamp than B.
**→ What each approach sacrifices:**
*   **Physical:** Sacrifices **correctness**; clock skew causes "Last-Write-Wins" (LWW) to silently drop data.
*   **Logical:** Sacrifices **real-world alignment**; you cannot know if an event happened "at lunch time" or "yesterday".
**→ Failure modes:** Physical clocks can lead to "causality violations" where an effect appears to happen before its cause.
**→ When to use it:** Use physical clocks for UI display or when uncertainty intervals are managed (e.g., Google Spanner’s TrueTime).
**→ When NOT to use it:** Never use physical clocks for ordering database writes unless the system can tolerate data loss.

---

### 4. Leadership Authority: Local Judgment vs. Quorum
**Problem:** How a node decides if it is still authorized to act as a leader.
**→ Why the naive solution fails:** A node might be **paused** (by GC or VM suspension) for longer than its lease duration. When it wakes up, it thinks it is still leader, unaware that time has passed and it has been replaced.
**→ Possible approaches:**
1. **Individual Node Judgment:** The node trusts its own clock and lease check.
2. **The Majority Rules (Quorum):** Decisions require a majority vote from multiple nodes.
**→ What each approach optimizes for:**
*   **Individual:** Optimizes for **latency**; the node can make fast local decisions without a network round-trip.
*   **Quorum:** Optimizes for **safety**; ensures the system remains in a valid state even if some nodes are delusional.
**→ What each approach sacrifices:**
*   **Individual:** Sacrifices **integrity**; leads to "split-brain" where two nodes both think they are leaders.
*   **Quorum:** Sacrifices **throughput**; every critical decision requires a network consensus.
**→ Failure modes:** Without quorums, "zombie" nodes can overwrite new data with old values. With quorums, the entire system stalls if a majority of nodes are unreachable.
**→ When to use it:** Use individual judgment for stateless, read-only caches. Use quorums for any state-changing operations or distributed locking.
**→ When NOT to use it:** Do not use individual judgment for leadership if "split-brain" can result in permanent data corruption.

---

### 5. Consistency Philosophy: Safety vs. Liveness (Availability)
**Problem:** Choosing how the system should react when its environmental assumptions (timing, network) are shattered.
**→ Why the naive solution fails:** You cannot have both perfect consistency and perfect availability during a network partition (CAP Theorem).
**→ Possible approaches:**
1. **Prioritize Safety:** "Nothing bad ever happens" (e.g., Linearizability).
2. **Prioritize Liveness:** "Something good eventually happens" (e.g., High Availability).
**→ What each approach optimizes for:**
*   **Safety:** Optimizes for **Integrity**; prevents contradictory data (e.g., no double-spending).
*   **Liveness:** Optimizes for **Availability**; ensures the system always returns a response even if it's potentially stale.
**→ What each approach sacrifices:**
*   **Safety:** Sacrifices **performance and availability**; the system must return an error if it can't guarantee the "truth".
*   **Liveness:** Sacrifices **consistency**; allows "temporary inconsistencies" that must be fixed later.
**→ Failure modes:** Over-prioritizing safety can lead to a system that is constantly offline due to minor network blips. Over-prioritizing liveness can lead to **perpetual inconsistency** if data corruption is never audited.
**→ When to use it:** Use Safety for financial systems of record and uniqueness constraints. Use Liveness for analytics or social feeds where an "apology" (like a slightly late post) is better than a "down" page.
**→ When NOT to use it:** Do not prioritize Liveness for systems where integrity violations are irreversible (e.g., discharging a medical device).