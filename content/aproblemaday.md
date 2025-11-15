+++
title = "A problem a day: Day 1"
    date = 2025-02-20
[taxonomies]
tags = ["algorithm", "a-problem-a-day"]
+++
## Origin

url: [Leetcode: Problem no. 2412](https://leetcode.com/problems/minimum-money-required-before-transactions/description/)
### Problem

You are given a **0-indexed** 2D integer array `transactions`, where `transactions[i] = [costi, cashbacki]`.

The array describes transactions, where each transaction must be completed exactly once in **some order**. At any given moment, you have a certain amount of `money`. In order to complete transaction `i`, `money >= costi` must hold true. After performing a transaction, `money` becomes `money - costi + cashbacki`.

Return _the minimum amount of_ `money` _required before any transaction so that all of the transactions can be completed **regardless of the order** of the transactions._

### Examples

**Input:** transactions = [ [2,1],[5,0],[4,2] ]
**Output:** 10
**Explanation:**
Starting with money = 10, the transactions can be performed in any order.
It can be shown that starting with money < 10 will fail to complete all transactions in some order.

**Input:** transactions = [ [3,0],[0,3] ]
**Output:** 3
**Explanation:**

- If transactions are in the order [ [3,0],[0,3] ], the minimum money required to complete the transactions is 3.
- If transactions are in the order [ [0,3],[3,0] ], the minimum money required to complete the transactions is 0.
Thus, starting with money = 3, the transactions can be performed in any order.

### Constraints

- 1 <= transactions.length <= $10^5$
- transactions[i].length == 2
- $0 <= cost_i, cashback_i <= 10^9$

## Journey

### Sorting

if i sort the outer array based on the cash back in ascending order, the order I get will require the most money initially(I think).
 It certainly works on the examples.

 Lets see
 for transactions of [ [4, 1], [3, 3], [5, 7], [9, 2] ]
 when sorted in ascending order of cashback
 sorted transaction = [ [4, 1], [9, 2], [3, 3], [5, 7] ]
 so minimum required cost: x - 4 + 1 - 9 + 2 - 3 + 3 - 5 = 0; x =  15

after calculating the money for all the permutations of the above transactions

```
[0]: ([4, 1], [3, 3], [5, 7], [9, 2]) initial money:  10
[1]: ([4, 1], [3, 3], [9, 2], [5, 7]) initial money:  15
[2]: ([4, 1], [5, 7], [3, 3], [9, 2]) initial money:  10
[3]: ([4, 1], [5, 7], [9, 2], [3, 3]) initial money:  11
[4]: ([4, 1], [9, 2], [3, 3], [5, 7]) initial money:  15
[5]: ([4, 1], [9, 2], [5, 7], [3, 3]) initial money:  11
[6]: ([3, 3], [4, 1], [5, 7], [9, 2]) initial money:  10
[7]: ([3, 3], [4, 1], [9, 2], [5, 7]) initial money:  15
[8]: ([3, 3], [5, 7], [4, 1], [9, 2]) initial money:  10
[9]: ([3, 3], [5, 7], [9, 2], [4, 1]) initial money:  9
[10]: ([3, 3], [9, 2], [4, 1], [5, 7]) initial money:  15
[11]: ([3, 3], [9, 2], [5, 7], [4, 1]) initial money:  9
[12]: ([5, 7], [4, 1], [3, 3], [9, 2]) initial money:  10
[13]: ([5, 7], [4, 1], [9, 2], [3, 3]) initial money:  11
[14]: ([5, 7], [3, 3], [4, 1], [9, 2]) initial money:  10
[15]: ([5, 7], [3, 3], [9, 2], [4, 1]) initial money:  9
[16]: ([5, 7], [9, 2], [4, 1], [3, 3]) initial money:  11
[17]: ([5, 7], [9, 2], [3, 3], [4, 1]) initial money:  9
[18]: ([9, 2], [4, 1], [3, 3], [5, 7]) initial money:  15
[19]: ([9, 2], [4, 1], [5, 7], [3, 3]) initial money:  11
[20]: ([9, 2], [3, 3], [4, 1], [5, 7]) initial money:  15
[21]: ([9, 2], [3, 3], [5, 7], [4, 1]) initial money:  9
[22]: ([9, 2], [5, 7], [4, 1], [3, 3]) initial money:  11
[23]: ([9, 2], [5, 7], [3, 3], [4, 1]) initial money:  9
```

The minimum amount of money for any order of transaction is 15 from the above results. Now 15 is returned by many orderings 1, 4, 7, 18, and 20. I don't know if this means my algorithm is correct or not but I'm gonna try it.

This sorting approach is fundamentally flawed but I'm gonna list some of the problems I thought there were with the approach that led me to the actual solution.

First the obvious one. The maximum initial money required could be at any point during the series of transactions and not only at the end. (I didn't think why when I was solving it last night., it was very late, otherwise I think I would have reached the solution sooner)
Okay so I should keep a maximum money that variable and update it only when money is greater than current maximum money. (again should have just thought for a moment why I needed to do this instead of running through hoops to make the solution work)

So I have the maximum initial money that I need throughout the transactions. This is it, right? Wrong. What about the ones where the cost is 0. Then the transaction is basically giving free cashback no matter if its smaller than the others. (I hadn't clocked it still but I'm getting there) I should just keep these transactions with zero cost at the end so that they don't affect the maximum_money.

"Okay hopefully it works now, I need to sleep" is what I thought but of course it doesn't and it didn't. Then I became aware of the third type of transactions in this whole thing. The transactions where cost is not 0 but some positive value but the cashback is still greater than cost so  it also decreases the maximum  initial money required and thus should be push to the last.

I had one final go where I put the transaction  with maximum cost among the negative (negative cost - cashback) at the front of those transactions. It still didn't work.

### Difference

And then I realized I didn't need to sort the transactions in the first place. Since only the transactions with the positive (in terms of cost) result mattered, I could find the sum of the positive transactions. Since I would have to pay the cost of transaction and then I got the cashback, if I added the highest cashback to that positive sum, then that is the minimum amount of money initially  to complete all of the transactions.

**If** the transactions where cost is less than cashback didn't exist, which practically shouldn't but the problem constraint  $0 <= cost_i, cashback_i <= 10^9$ doesn't say anything about that. So the last step of the positive sum is to negate the effect of the highest cashback since I don't get it until I pay the cost of that transaction.

Now I should include it in if the highest cost among the negative transactions(where cashback > cost) is greater than highest cashback. This lead me to the solution below.

```rust
impl Solution {
    pub fn minimum_money(transactions: Vec<Vec<i32>>) -> i64 {
        let mut positive_sum: i32 = transactions.iter()
            .map(|transaction| transaction[0] - transaction[1])
            .filter(|&diff| diff > 0)
            .sum();
        let mut highest_neg = 0;
        for tran in &transactions {
            if tran[0] < tran[1] && tran[0] > highest_neg {
                highest_neg = tran[0];
            }
        }
        let mut highest_cashback_index = 0;
        let mut highest_cashback = 0;
        for (i, tran) in transactions.iter().enumerate() {
            if (tran[0] > tran[1] && highest_cashback < tran[1]) {
                highest_cashback = tran[1];
                highest_cashback_index = i;
            }
        }

        if highest_cashback < highest_neg {
            (positive_sum+highest_neg) as i64
        } else {
            (positive_sum+highest_cashback) as i64
        }
    }
}
```

It didn't work because I forgot to factor in the transactions where cost == cashback. It was very late at night so I went to sleep which I couldn't do anyway. I kept thinking about it and not long before I realized about the zero transactions.

## Destination

Each transactions can be classified into two types, positive transactions where cost is greater than cashback and negative ones where cost is equal to or less than cash back.

Let’s just look at the positive transactions for now—where the cost is more than the cashback. Since we always have to pay more than we get back, the minimum amount of money we need keeps going up with each transaction. But since we get the cashback right after each transaction, the order doesn’t really matter—except for the last one.

The last transaction is different because we need enough money to cover its cost before we get its cashback. So, the worst possible order—the one that makes us need the most money upfront—is when the transaction with the highest cashback is saved for last. If we got that cashback earlier, it would’ve helped lower the amount of money we needed. But by putting it last, we miss out on that advantage, making us start with more money to complete every transactions.

For negative transactions, we only need to consider the one with the highest cost. Any other negative transaction would give more cashback than it costs, meaning it would actually reduce the amount of money we need upfront.

If $M_p$ is the minimum money required for the positive transactions, then adding the most expensive negative transaction with cost $C_n$ means we also need to complete the last positive transaction. Completing the last positive transaction gives us its maximum cashback, $B_p$.

So, the initial money required in this case would be $M_p + C_n - B_p$. Since we're trying to find the worst-case scenario—the order that requires the most money upfront—we need to check if including this negative transaction actually increases the required money or not. If $C_n$ is greater than $B_p$, it makes the starting money go up, meaning it should be included in the worst case. If not, then we can ignore it because it doesn’t increase the initial money required.

This is the final solution I came up with. It is in rust.

```rust
pub fn minimum_money(transactions: Vec<Vec<i32>>) -> i64 {
 let mut pos_sum: i64 = 0;
 let mut max_cb = 0;
 let mut max_neg = 0;
 for tran in transactions {
  let diff = tran[0] - tran[1];
  if (diff > 0) {
   pos_sum += diff as i64;
   if max_cb < tran[1] {
    max_cb = tran[1];
   }
  } else {
   if max_neg < tran[0] {
    max_neg = tran[0];
   }
  }
 }

 if max_cb < max_neg {
  pos_sum+max_neg as i64
 } else {
  pos_sum+max_cb as i64
 }
}
```
