+++
title = "A problem a day: Day 2"
date = 2025-02-22
[taxonomies]
tags = ["algorithm", "a-problem-a-day"]
+++
## Origin

url: [Leetcode: Problem no. 55](https://leetcode.com/problems/jump-game/description/)

### Problem

You are given an integer array `nums`. You are initially positioned at the array's **first index**, and each element in the array represents your maximum jump length at that position.

Return `true` _if you can reach the last index, or_ `false` _otherwise_.

### Examples

**Input:** nums = [2,3,1,1,4]
**Output:** true
**Explanation:** Jump 1 step from index 0 to 1, then 3 steps to the last index.

**Input:** nums = [3,2,1,0,4]
**Output:** false
**Explanation:** You will always arrive at index 3 no matter what. Its maximum jump length is 0, which makes it impossible to reach the last index.

### Constraints

- $1 <= nums.length <= 10^4$
- $0 <= nums[i] <= 10^5$

## Journey

Well if the index of the array are the nodes of a graph and the jumps indicate the path from one node to another ten its just finding if a path exists from first to last node(index). With the added context and information I feel like there is an easier way to do this.

### A Simple DFS

- put the start index(0) in the stack.
- while the stack is not empty
  - pop from stack
  - if this is the final index (nums.length) return true
  - put all of its children (current index + (1 to the value at current index))
- return false

DFS time complexity is $\mathcal{O}(b^m)$ where b is the branching factor and m is the maximum depth
which is a lot to be honest.
 And I was correct.
 ![Solution rejected due to time limit exceeding](/images/apad2-time.png)

so I do need to optimize this using the added information or go for a different approach.

starting from the last index, move down wards to the first index
if value at current index is greater than or equal to (goal index - current index) set goal index to current index

when the loop ends, if the goal index is 0 return true else false

I don't know how I went from exponential time to linear time. Also I don't know what made me think about the numbers in reverse.

## Destination

We have a list of numbers, for example like this [2,3,1,1,4], and we want to find if we can go from index 0 to last index. Say we are at the goal index, so without any jumping we are already at the goal index and thus have reached it.

What about the index one before the goal. If we can jump from this one to goal, we have reached the goal index. How do we know if we can jump from this one to the last one? Since the value at that index indicates the jumping ability, we can jump from current index to goal index if the value is greater than the distance between current index and goal index. So if the value is greater than or equal to 1 we can reach the goal index from the index one before it.

Do you know what this means? It means the if we can somehow reach this index, then we can reach the goal index as well. So we can just set this index as the goal index.

What if we can't jump from this index to the goal one? Does that mean we stop the algorithm and return false? No, because the index before this index may have the jumping ability to jump over this index. Imagine the numbers as [2, 0, 1]. The second last index's value is 0 so we can't reach the goal state from it. But the index before it, the first index,  has value 2 which is equal to the distance between first index and the goal index. So we set the goal index to first index.

We continue this until we reach the first index. At this point, the goal index will the index closest to the first index from which we can reach the actual goal index defined in the problem. If we can reach  the current goal index from the first index, the goal index becomes the first index.

If ,after the loop is finished, the goal index is the current index, that means we can reach the actual goal index from the first index.

```rust
impl Solution {
    pub fn can_jump(nums: Vec<i32>) -> bool {
        let mut goal_idx = nums.len() - 1;
        for (curr_idx, val) in nums.iter().enumerate().rev() {
            if *val >= (goal_idx - curr_idx) as i32 {
                goal_idx = curr_idx;
            }
        } 
        if goal_idx == 0 {
            return true;
        }
        return false;
    }
}
```
