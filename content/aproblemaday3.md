+++
title = "A problem a day: Day 3"
date = 2025-02-23
[taxonomies]
tags = ["algorithm"]
+++

## Origin

url: [Leetcode: Problem no. 483](https://leetcode.com/problems/smallest-good-base/description/)

### Problem

Given an integer `n` represented as a string, return _the smallest **good base** of_ `n`.
We call `k >= 2` a **good base** of `n`, if all digits of `n` base `k` are `1`'s.

### Examples

#### Example 1

- **Input:** n = "13"
- **Output:** "3"
- **Explanation:** 13 base 3 is 111.

#### Example 2

- **Input:** n = "4681"
- **Output:** "8"
- **Explanation:** 4681 base 8 is 11111.

#### Example 3

- **Input:** n = "1000000000000000000"
- **Output:** "999999999999999999"
- **Explanation:** 1000000000000000000 base 999999999999999999 is 11.

### Constraints

- `n` is an integer in the range $[3, 10^{18}]$.
- `n` does not contain any leading zeros.

## Journey

I'd like to believe there is a formula for this.
Every digit should be 1 so
$$
n = \sum_{p=0}^t k^p
$$
I need to find out what k is. I don't know that `t` gives the smallest `k` for a given `n`. I do know that for `t=1`
$n = k^0 + k^1$
$k = n-1$
so t = 1 gives the largest (and is some cases like example 3 above the only) base `k` for which all the digits are 1.

I also know that as I keep increasing t, then I will keep finding a `k` smaller than the previous one (if it exists).
But where do I stop? or better yet where do I begin high and keep decreasing `t` until I find a valid `k`
since $k \ge 2$, we can find the largest t where k is 2.
$$
\begin{align*}
n &= \sum_{p=0}^t 2^p \\\\
n &= 2^0 + 2^1 + 2^2 + ... + 2^t \\\\
n &= \frac{1-2^{t+1}}{1-2} \\\\
n &= 2^{t+1} - 1 \\\\
t+1 &= \log_2(n+1) \\\\
t &= \log_2(\frac{n+1}{2})
\end{align*}
$$
so we can start at `t` equal to the above value, and `k=2` we can compute $n\prime$ as
$$
n\prime = \frac{k^{t+1} -1}{k-1}
$$
Now if this $n\prime$ is equal to the given n then we have found that the good base `k`. If not, then there are two cases. If $n\prime \le n$, then we should increment `k` otherwise decrement `t`. Now the solution seems innocent enough.

Remember that in the worst case we have to go from `k=2` go k = `n-1` and thats not all. We are iterating over `t` which starts at the log of `n+1` and stops at `t=2`. For simplification, lets say that we check for all values of `k` in range [2, n-1], for all values of `t` in range $[log_2{(n+1)/2}, 2]$. Then the worst case time complexity is $\mathcal{O}(n log(n))$.

Which doesn't seem much. The sorting algorithms are $\mathcal{O}(n log(n))$.  But look at the constraints. The n can be as high as $10^{18}$.  Now my laptop has a cpu with speed of 2GHz. (forget about how many processors ans such for now). That means that it can run a maximum of $2*10^9$ clock cycles per second(also forgetting about boost and overclock because I also don't know). Lets be very very very optimistic here and suppose that every operation in that algorithm takes a single clock cycle[_I may not know hardware stuff very much but I did have microprocessor as a subject in one of semester. So trust me when I say this would have been the biggest lie I've ever told if it wasn't for the next line._]. Lets also suppose that the line of code translates 1:1 to operations then the code below (in python).

```python
def good_base(n: float):
    t = math.floor(math.log2((n+1) / 2))
    k = 2
    while True:
        n_prime = (math.pow(k, t+1) - 1) / (k-1)
        if n_prime == n:
            break
        elif n_prime < n:
            k+=1
        else:
            t-=1
    return k
```

Then the loop of 8 lines takes about $\frac{8n}{2*10^{9}}$ seconds to run. For numbers less than $10^9$, it takes less than a second. Why bother with the numbers in between, lets go straight to the last one. It takes about $4000000000$ seconds. That is, it takes $126.84$ years for this hugely oversimplified and optimistic imaginary computational model to find out that the required k is $10^{18} -1$.

So what to do. what to do.

aha. Good old binary search to the rescue
why do i have to search the the numbers for k 2 through n-1. what i can do is
check for every t from the maximum one given by $\log_2(\frac{n+1}{2})$ down to 1
if k = 2 results in $n\prime = n$. if it does we have found the smallest good base if not we have to go for the middle one. so what is the middle one
$$
k_{mid} = \frac{k_0(2)+?}{2}
$$
we need to figure out the maximum k. Now could just use `n-1`. if we are doing binary search then we only check the log(n) numbers.
$log_2(10^{18}) = 59.79$
and we have to check that max t to 1 and max t is also given by log base 2 of n. So the number of elements we are checking for the maximum n defined for this problem is nearly $60*60=3600$. compare that with $60 * 10 ^{18}$ we were doing earlier.
Lo and behold

```python
import math
def good_base2(n):
    if n < 2:
        return 1
    t = 0
    temp = (n + 1) // 2
    while temp > 1:
        temp //= 2
        t += 1
    found = False
    while not found and t >= 0:
        k = 2
        k_m = n
        prev_k = 1
        while k < n:
            numerator = pow(k, t + 1) - 1
            denominator = k - 1
            n_prime = numerator // denominator            
            if n_prime == n:
                found = True
                break
            elif n_prime < n:
                prev_k = k
                k = (k + k_m) // 2
            else:
                k_m = k
                k = (prev_k + k_m) // 2
            if prev_k == k:
                t -= 1
                break
    return k 
```

an imaginary and hopefully delusional time of 126 years to
 ![Running time of solution 2 to n = 10^18](/images/apad3-sol2-time.png)

but we can do better. atleast theoretically. It does involve computing square root.
What am I taking about?
well you see how we computed the maximum t given a n.
we can also do the opposite. we can compute the maximum `k` for a given `t` such that any value of k more than this will make `n prime` greater than `n`.
we have
$$
n = k^0 + k ^1 + k^2 + .... + k^t
$$
lets us ignore all the expression of `k` before $k^t$ so it becomes
$$
\begin{aligned}
n = k^t \\
k = \sqrt[t]{n}
\end{aligned}
$$
for any k greater than this would make the calculated n prime greater than n.
as a bonus we can just check with the final value as $\sqrt[t]{n}$ and if the calculated n is smaller than given n then we just go to the next t. What if its bigger than the given then we just subtract one from the calculated `k` for this t ($\sqrt[t]{n}$).

```python
def good_base3(n):
    if n < 2:
        return 1
    t = 0
    temp = (n + 1) // 2
    while temp > 1:
        temp //= 2
        t += 1
    k = math.floor(n ** (1/t))
    while t > 0:
        numerator = pow(k, t + 1) - 1
        denominator = k - 1
        n_prime = numerator // denominator
        if (n_prime == n):
            break
        if (n_prime > n):
            k -= 1
        else:
            t-=1
            k = math.floor(n ** (1/t))
    return k 
```

 ![Running time of solution 3 to n = 10^18](/images/apad3-sol3-time.png)

And I wanted to calculate how many iterations each one took.
The `good_base2` took a total of 3511 iterations while the `good_base3` took 62 iterations. This make me not sure about the correctness of the last one. I mean I could just try it on leetcode and see. Also I have been avoiding the String thing in the questions. I'm assuming I can just parse the string as u128 in rust.
okay second one is definetely wrong.
well it wasn't wrong the floats caused the error. if t ever becomes 1, the float power function to calculate k cannot
store the value exactly. (this was the problem in both rust and python). rust has f128 which didn't cause this issue but thats not a stable feature so i could not use that.
anyway if it t becomes 1 then we know the answer is n-1 so i run the loop until t is greater than 1.
when the loop exit, I know if it exited by finding the value or if t reaching one. I used bool found for that but as im writing this I realize I could have just checked t is 1 or not.
also python solution also caused the floating point approximation error. so does that mean python stores floating point as 64 bits too.

and why didn't this happen for the number $10^{18}$ but for a complete random number.
I think this could all be explained if I made some proof and explaination of the problem but then again I'm really not in the mood. so

## Destination

I don't really know what to write here. I mean usually I write how the solution works but for this one I feel like I've already explained everything in the [[#Journey]] section above. So have a look at the solution in rust.

```rust
impl Solution {
    pub fn smallest_good_base(n: String) -> String {
        let n = n.parse::<u128>().unwrap();
        let mut temp = (n + 1) / 2;
        let mut t = 0;
        let mut found = false;
        // approximate log base 2
        while temp > 1 {
            temp /= 2;
            t += 1;
        }
        let mut k = ((n as f64).powf(1.0 / t as f64)).floor() as u128;
        while t > 1 {
            let n_prime = (k.pow(t + 1) - 1) / (k - 1);
            if n_prime == n {
                found = true;
                break;
            } else if n_prime > n {
                k -= 1;
            } else {
                t -= 1;
                k = ((n as f64).powf(1.0 / t as f64)).floor() as u128;
            }
        }
        if found {
            k.to_string()
        } else {
            (n-1).to_string()
        }
    }
}
```
