---
title: "Understanding Queues with Swift: A Real-World Example"
date: "2025-06-24"
excerpt: "Learn how queues work in programming using a real-world example of waiting in line at Olive Garden, with practical Swift implementations."
tags: ["swift", "data-structures", "queues"]
---
The other day, I visited Olive Garden and found myself waiting in line for a table. I was fourth in the queue. The waiter explained that I’d receive a message once it was my turn, as they operated on a **first-come, first-served** basis. That’s when I remembered—queues in programming function exactly the same way as lines in real life!

Unlike arrays, you cannot access random elements in a queue. Instead, a queue supports two primary operations:

* `enqueue`: Add an item to the end of the queue.
* `dequeue`: Remove an item from the front of the queue.

When you `enqueue` two items to the list, the first item you added will be `dequeued` before the second item. This behavior is why a queue is called a **FIFO** (First-In, First-Out) data structure.

> Returning to our real-life example, if another person arrives at Olive Garden after me, they will be enqueued behind me and receive queue number 5. When the waiter dequeues, the person at the front of the line (queue number 1) will receive a message indicating it’s their turn to be served.

### What can we put into a queue?

When we say we ‘enqueue an item,’ we’re talking about adding a piece of data into the queue. This data could be something simple—like a number `44`, a word `"Hello"`, or even something complex—such as an object that represents an entire book, complete with chapters, titles, and authors.

### Example

Let’s implement a simple example based on our real-life scenario.
We’ll store the names of people waiting in line at Olive Garden.

```swift
// Our First In, First Out data structure for tracking people in line
struct Queue {

     // The underlying array that stores the names of the people in queue.
    private var names = [String]()

    // Adds name to the back of queue
    mutating func enqueue(_ name: String) {
        self.names.append(name)
    }

    // Removes and returns the name at the front of the queue.
    mutating func dequeue() -> String {
        return self.names.removeFirst()
    }
}

// Creating our queue object
var oliveGardenQueue = Queue()

// Adding customers to the line
oliveGardenQueue.enqueue("John")
oliveGardenQueue.enqueue("Jane")
oliveGardenQueue.enqueue("Alice")
oliveGardenQueue.enqueue("Bob")

// Now assume waiters calls first in line
print(oliveGardenQueue.dequeue()) // Output: John

// Lets call another one in line
print(oliveGardenQueue.dequeue()) // Output: Jane
```
As you may have noticed, our implementation has a few limitations:
1.	It only stores names—in other words, plain string values.
2.	It doesn’t handle the case where we try to dequeue from an empty queue. If we call dequeue() when the list is empty, the program will crash.

### Lets revise it
First, let’s make our queue more generic, so it can store any type of data—not just strings.
We’ll also update the dequeue method to return an optional value. That way, if the queue is empty, it will return nil instead of crashing.

In addition, we’ll add a few useful methods:
- `peek` — to look at the front item without removing it
- `size` — to check how many items are in the queue
- `isEmpty` — to quickly check if the queue is empty

These additions will make our queue more flexible and convenient to use.

```swift
struct Queue<T> {
    private var elements: [T] = []

    // Add an element to the queue
    mutating func enqueue(_ element: T) {
        elements.append(element)
    }

    // Remove and return the first element of the queue
    mutating func dequeue() -> T? {
        guard !elements.isEmpty else { return nil }
        return elements.removeFirst()
    }

    // View the first element without removing it
    func peek() -> T? {
        return elements.first
    }

    // Check if the queue is empty
    func isEmpty() -> Bool {
        return elements.isEmpty
    }

    // Get the size of the queue
    func size() -> Int {
        return elements.count
    }
}

var queue = Queue<Int>()

queue.enqueue(10)
queue.enqueue(20)
queue.enqueue(30)

print(queue.peek() ?? "Queue is empty") // Output: 10

print(queue.dequeue() ?? "Queue is empty") // Output: 10
print(queue.dequeue() ?? "Queue is empty") // Output: 20

print(queue.size()) // Output: 1
print(queue.isEmpty()) // Output: false
```

### When to use Queues?
Queues are useful when you need to manage elements in sequential order, such as:
- Task scheduling
- Handling requests (e.g., print queue, web requests)
- Managing operations in order of arrival

Queues are also used in some algorithms. For example, the Breadth-First Search (BFS) algorithm relies on a queue to keep track of which nodes to visit next.

### Final notes
In this example, we used an array as the underlying data structure to store elements. However, we could also use a linked list for this purpose.

Queues are a simple but powerful concept that show up in many real-world systems. Now that you understand the basics, try implementing your own queue from scratch or explore how queues are used in more advanced topics like concurrency, simulations, or graph algorithms. Practice is the best way to deepen your understanding!
