---
title: "Understanding Snowflake IDs with Swift"
date: "2025-07-03"
excerpt: "Learn how snowflake IDs work in programming using a real-world example of waiting in line at Olive Garden, with practical Swift implementations."
tags: ["swift", "data-structures", "snowflake-id"]
---

## Introduction
> "How can millions of computers around the world each hand out a brand‑new, perfectly unique number without talking to a central server first?"

Modern applications from social networks to e-commerce platforms need a reliable way to generate unique identifiers (IDs) for things like users, posts, orders or log entries. In a single server app you might simply generate IDs by incrementing a counter (e.g., 1, 2, 3, ...), but this approach has limitations when scaling to multiple servers, or dozens of data-centers or thousands of microservices that must tag millions of events every second?

Meet Snowflake IDs - a ingenious 64-bit identifier scheme originally open-source by Twitter (now X). It allows every machine to generate IDs locally while still ensuring global uniqueness, time ordered, and compact.

**In this short tutorial we’ll:**
- Demystify the 64‑bit layout (timestamp, node IDs, sequence counter).
- Build a concurrency‑safe Swift generator using actors.
- Decode and inspect the parts of a generated ID.
- Highlight limitations (sequence overflow, clock drift) and best‑practice mitigations.

### What is a Snowflake ID?

A Snowflake ID is a 64-bit integer that consists of the following key pieces of information in a compact form:
- **Timestamp** - the exact moment the ID was generated, stored in milliseconds since a custom epoch.
- **Node ID** - a unique identifier for the data center and machine generating the ID. 
- **Sequence** - a per‑millisecond counter that guarantees uniqueness even if the same node issues thousands of IDs in the same millisecond.

Because the timestamp occupies the highest bits, newer IDs are always greater than older ones. Meanwhile splitting the remaining bits between node and sequence field lets every machine generate IDs independently without the need of central database lock or network round trips.

### Anatomy of a 64-bit Snowflake ID

Below is a classic layout of a 64-bit Snowflake ID:

![Snowflake ID Layout](/blog/snowflake_struct.png)


| Field             | Bits | Range              | What it does                                                         |
| ----------------- | ---- | ------------------ | -------------------------------------------------------------------- |
| **Timestamp**     | `41`   | ≈2^41 ms ≈ 69 years | Milliseconds since your **custom epoch** (not UNIX‑epoch).           |
| **Datacenter**    | `5`    | 0–31               | Identifies the physical/logic datacenter generating the ID.          |
| **Worker / Node** | `5`    | 0–31               | Distinguishes workers/processes inside the same datacenter.          |
| **Sequence**      | `12`   | 0–4095 per ms      | Incremented if the same node issues multiple IDs in one millisecond. |

Because the timestamp occupies the most‑significant bits, sorting IDs numerically also sorts them chronologically. The remaining `22 bits` (5 + 5 + 12) ensure uniqueness across nodes and bursts of traffic.

> **Why milliseconds?** Using milliseconds (rather than seconds) gives much finer granularity and lets each node generate up to `4096` IDs per millisecond without collisions.

### Implementing a Snowflake ID Generator

Now we are going to implement a Snowflake ID generator in Swift. We will use actors to ensure thread safety and prevent race conditions.

```swift

actor SnowflakeActor {
    /// Number of bits dedicated to each field
    private static let timestampBits: UInt8  = 41
    private static let datacenterBits: UInt8 = 5
    private static let workerBits: UInt8     = 5
    private static let sequenceBits: UInt8   = 12

    /// Maximum value for the sequence field
    private static let maxSequence: UInt64 = (1 << sequenceBits) - 1

    /// Shift amounts (stored as Int for easy bit‑shifts)
    private let workerShift:     Int
    private let datacenterShift: Int
    private let timestampShift:  Int

    // Custom epoch (2025‑07‑06T00:00:00Z) expressed in **milliseconds**
    private let epoch: UInt64 = 1_751_760_000_000
    private let datacenterId: UInt64
    private let workerId: UInt64

    /// Mutable state protected by actor isolation
    private var lastTimestamp: UInt64 = 0
    private var sequence: UInt64 = 0

    init(datacenterId: UInt64, workerId: UInt64) {
        precondition(datacenterId < 32 && workerId < 32, "IDs must be between 0 and 31 (inclusive)")
        self.datacenterId = datacenterId
        self.workerId = workerId

        self.workerShift     = Int(Self.sequenceBits)
        self.datacenterShift = Int(Self.sequenceBits + Self.workerBits)
        self.timestampShift  = Int(Self.sequenceBits + Self.workerBits + Self.datacenterBits)
    }

    /// Generates the next unique Snowflake ID.
    func nextID() async -> Int64 {
        var ts = currentTimeMillis()

        // Handle clock rollback
        if ts < lastTimestamp {
            ts = await waitUntil(after: lastTimestamp)
        }

        // Same millisecond → increment sequence
        if ts == lastTimestamp {
            sequence = (sequence + 1) & Self.maxSequence // wrap to 0…4095
            if sequence == 0 {
                // Sequence overflow → move to next millisecond
                ts = await waitUntil(after: lastTimestamp)
            }
        } else {
            sequence = 0
        }
        lastTimestamp = ts

        let raw: UInt64 =
              ((ts - epoch) << timestampShift) |
              (datacenterId << datacenterShift) |
              (workerId << workerShift) |
              sequence

        return Int64(bitPattern: raw) // Sign‑bit is 0 → always positive
    }

    private func currentTimeMillis() -> UInt64 {
        UInt64(Date().timeIntervalSince1970 * 1000)
    }

    private func waitUntil(after target: UInt64) async -> UInt64 {
        var ts = currentTimeMillis()
        while ts <= target {
            await Task.yield() // Give up the executor slice
            ts = currentTimeMillis()
        }
        return ts
    }
}

```

`sequence = (sequence + 1) & Self.maxSequence` - if this code bit confusing let me explain it in detail, this is a way to make sure the sequence doesn't overflow, and stays within the bounds of the sequence field, we could have used modulo operator but bit-wise operation is faster and more efficient.

#### Usage Example

```swift
let snowflake = SnowflakeActor(datacenterId: 1, workerId: 1)

for _ in 1...10 {
    let id = try! await snowflake.nextID()
    print(id)
}
```

Because SnowflakeActor is an actor, every call to nextID() is automatically serialised inside the actor’s isolated context, eliminating race conditions without needing any locks or mutexes.

### Decoding a Snowflake ID

Snowflake IDs aren't just numbers, you can extract useful information from them, such as the timestamp, datacenter ID, worker ID, and sequence number.

Now let's decode a Snowflake ID:

```swift
struct SnowflakeParts {
    let timestamp: Date       // Original Date
    let datacenterId: UInt64
    let workerId: UInt64
    let sequence: UInt64
}

func decodeSnowflake(_ id: Int64, epoch: UInt64 = 1_751_760_000_000) -> SnowflakeParts {
    let raw = UInt64(bitPattern: id)       // treat it as unsigned bits

    let timestampBits: UInt8  = 41
    let datacenterBits: UInt8 = 5
    let workerBits: UInt8     = 5
    let sequenceBits: UInt8   = 12

    let sequenceMask: UInt64   = (1 << sequenceBits) - 1
    let workerMask: UInt64     = (1 << workerBits) - 1
    let datacenterMask: UInt64 = (1 << datacenterBits) - 1

    let sequence   = raw & sequenceMask
    let workerId   = (raw >> sequenceBits) & workerMask
    let datacenter = (raw >> (sequenceBits + workerBits)) & datacenterMask
    let timestamp  = (raw >> (sequenceBits + workerBits + datacenterBits)) + epoch

    return SnowflakeParts(
        timestamp: Date(timeIntervalSince1970: TimeInterval(timestamp) / 1000),
        datacenterId: datacenter,
        workerId: workerId,
        sequence: sequence
    )
}

```

#### Usage Example

```swift
let gen = SnowflakeActor(datacenterId: 1, workerId: 4)
let id  = await gen.nextID()

let parts = decodeSnowflake(id)
print(parts.timestamp)      // exact Date object
print(parts.datacenterId)   // 1
print(parts.workerId)       // 4
print(parts.sequence)       // 0…4095
```

### Limitations

Snowflake IDs have a few limitations:

- The timestamp field is limited to 41 bits, which means it can only represent dates for about 69 years from the custom epoch.
- You are limited to having 32 datacenters and 32 workers per datacenter.
- The sequence field is limited to 12 bits, which means it can only represent 4096 IDs per millisecond.

### Handling Clock Drift

Snowflake relies on the system clock. If a machine’s clock jumps backwards, new IDs could numerically collide or appear older than they are. Strategies:

- Wait until the clock catches up. Like in the example above, we wait until the clock catches up and then generate the next ID. This is simple and commonly used, but can stall ID generation for the duration of the clock drift.
- Throw an exception, this approach forces external clock discipline and callers have to handle the exception.
- Bump datacenter ID/worker ID, dynamically switch to spare node ID when drift is detected. This is a more complex approach and needs a pool of spares nodes/datacenter IDs.

For most servers synced with NTP, brief waits are fine. If you need high-frequency ID generation consider using "throw" strategy and enforce strict timekeeping.

### Pros & Cons Recap

#### Pros
- Globally unique without a central authority
- Monotonically increasing -> great for sorting and indexing
- Compact `8` bytes
- Works offline / partition-tolerant

#### Cons
- Requires trust in the system clock
- Sequence limit of `4096` IDs per millisecond per node
- Predictable ID values -> not suitable for security‑through‑obscurity.

### Final Thoughts

Snowflake IDs are a powerful tool for generating unique identifiers in distributed systems. They are compact, globally unique, and monotonically increasing, making them ideal for use in distributed systems. You can awlays adjust the layout of the ID to suit your needs. Perhaps you can decrease numbers of sequence bits to increase bits for datacenter or worker ID. Or even drop one of the fields entirely if you don't need it. To take advantage of the swift language you can create a custom type to represent the ID. 


