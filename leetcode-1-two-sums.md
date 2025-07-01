---
title: "Two Sum – Explained Step by Step (Swift)"
date: "2025-06-29"
excerpt: "A clear and beginner-friendly breakdown of the LeetCode Two Sum problem, with brute-force and optimized Swift solutions using hash tables."
tags: ["swift", "algorithms", "hash-table", "array", "LeetCode", "two-sum", "easy", "problem-solving", "interview", "data-structure"]
---

## Problem Statement

> Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to target.
>You may assume that each input would have **exactly one solution**, and you may not use the same element twice.
>You can return the answer in any order.

Link to the problem: [Two Sum](https://leetcode.com/problems/two-sum/)

### Example

```swift
let nums = [2, 7, 11, 15]
let target = 9
let result = twoSum(nums, target)
print(result) // Output: [0, 1]
```

### Constraints

```swift
nums.count >= 2
nums.count <= 10^4
-10^9 <= nums[i] <= 10^9
-10^9 <= target <= 10^9
```

## Problem breakdown
Let's break down the problem statement into smaller chunks.

1. We are given an array of integers `nums` and an integer `target`.
2. We need to return indices of the two numbers such that they add up to target.
3. You may assume that each input would have **exactly one solution**, and you may not use the same element twice.
4. You can return the answer in any order.

## Solution
When we first approach this problem, we might try checking all possible pairs of numbers to see if any add up to the target.

### How it works
- Loop through every element in the array.
- For each element, loop through the rest of the array to check if there’s another number that adds up to the target.

### Code in swift
```swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    for i in 0..<nums.count {
        for j in i+1..<nums.count {
            if nums[i] + nums[j] == target {
                return [i, j]
            }
        }
    }
    // Although the problem states that there is exactly one solution, we are going to return an empty array if no solution is found.
    return []
}
```

### Time & Space Complexity
Let's analyze the time & space complexity of the solution. 

- The outer loop runs `n` times, where `n` is the length of the input array `nums`.
- The inner loop also runs `n` times for each iteration of the outer loop.

Therefore, the time complexity of the solution is O(n^2), where `n` is the length of the input array `nums`.

- The space complexity of the solution is O(1), as we are not using any additional space that scales with the input size.

## Optimized Solution
There is a way to optimize the solution to O(n) time complexity.

### How it works
- Use a hash map to store the indices of the numbers we have seen so far.
- Loop through the array and for each number, check if the target minus the number is in the hash map.
- If it is, return the indices of the two numbers.
- If it is not, add the number and its index to the hash map.

### Code in swift
```swift
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var map = [Int: Int]()
    for i in 0..<nums.count {
        let complement = target - nums[i]
        if let j = map[complement] {
            return [j, i]
        }
        map[nums[i]] = i
    }
    // Although the problem states that there is exactly one solution, we are going to return an empty array if no solution is found.
    return []
}
```

### Time & Space Complexity
Let's analyze the time & space complexity of the solution.

There is only one loop that runs `n` times, where `n` is the length of the input array `nums`.

Therefore, the time complexity of the solution is O(n), where `n` is the length of the input array `nums`.

This time, the space complexity of the solution is O(n), as we are using a hash map to store the indices of the numbers we have seen so far.

## Key Takeaways
- When solving the problem, start with a brute-force solution. Understand the problem.
- When you find a solution, you understand the problem better, then you can try to find patterns or data structures (like hash maps) that make things faster.
- The hash map trick is one of the most powerful and reusable techniques in algorithm problems!

## Recommendation
- Try to modify the code, play with it, and see what happens.
- Always come back to the problem after a while and try to solve it again.
- Our brain works in patterns, when you practice solving problems, you start to recognize patterns and can solve problems faster.


