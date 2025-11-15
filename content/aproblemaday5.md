+++
title = "A problem a day: Day 5"
date = 2025-02-25
[taxonomies]
tags = ["algorithm", "a-problem-a-day"]
+++

## Origin

### Problem

The **appeal** of a string is the number of **distinct** characters found in the string.

- For example, the appeal of `"abbca"` is `3` because it has `3` distinct characters: `'a'`, `'b'`, and `'c'`.
Given a string `s`, return _the **total appeal of all of its substrings.**_
A **substring** is a contiguous sequence of characters within a string.

### Examples

#### Example 1

**Input:** s = "abbca"
**Output:** 28
**Explanation:** The following are the substrings of "abbca":

- Substrings of length 1: "a", "b", "b", "c", "a" have an appeal of 1, 1, 1, 1, and 1 respectively. The sum is 5.
- Substrings of length 2: "ab", "bb", "bc", "ca" have an appeal of 2, 1, 2, and 2 respectively. The sum is 7.
- Substrings of length 3: "abb", "bbc", "bca" have an appeal of 2, 2, and 3 respectively. The sum is 7.
- Substrings of length 4: "abbc", "bbca" have an appeal of 3 and 3 respectively. The sum is 6.
- Substrings of length 5: "abbca" has an appeal of 3. The sum is 3.
The total sum is 5 + 7 + 7 + 6 + 3 = 28.

#### Example 2

**Input:** s = "code"
**Output:** 20
**Explanation:** The following are the substrings of "code":

- Substrings of length 1: "c", "o", "d", "e" have an appeal of 1, 1, 1, and 1 respectively. The sum is 4.
- Substrings of length 2: "co", "od", "de" have an appeal of 2, 2, and 2 respectively. The sum is 6.
- Substrings of length 3: "cod", "ode" have an appeal of 3 and 3 respectively. The sum is 6.
- Substrings of length 4: "code" has an appeal of 4. The sum is 4.
The total sum is 4 + 6 + 6 + 4 = 20.

### Constraints

- $1 <= s.length <= 10^5$
- `s` consists of lowercase English letters.

## Journey

To find the appeal of just the string, loop through every element, putting each of them in a set. finally calculate the length of set.
do this for every substring of length 1 to n

How many substrings does the string have.
there are n substrings of size 1
n-1 substrings of size 2 and so on until 1 substring of size 1
so total substrings = $\frac{n(n+1)}{2}$
and we have to check for unique elements for each substring
so the time complexity is $\mathcal{O}(n^3)$

so is there a better way. There has to be right?
the previous loop should help then next one

I couldn't find a better one for some time so i just tried the above process (code below) and yea time limit exceeded so there is a better solution. I knew there was but i just had to be sure after [yerterday](https://faulty.carboxi.de/aproblemaday4)'s disaster of a question.

```rust
use std::collections::HashSet;
impl Solution {
    pub fn appeal_sum(s: String) -> i64 {
        let s = s.as_bytes();
        let n = s.len();
        let mut total_appeal = n as i64;
        for size in 1..n+1 {
            for i in 0..n-size {
                let mut set: HashSet<u8> = HashSet::with_capacity(26);
                for j in i..i+size+1 {
                    set.insert(s[j]);                  
                }
                total_appeal += set.len() as i64;
            }
        }
        total_appeal
    }
}
```

anyway so what is it. what can i do.

divide and conquer??

let me try something

I tried using python to calculate the appeal of a substring to see if it worked. well the time exceeded on this one too. but on a different and bigger size string.

```python
class Solution:
    def appealSum(self, s: str) -> int:
        n = len(s)
        total_appeal = n
        for i in range(1, n+1):
            for j in range(n-i):
                total_appeal += len(set(s[j:j+i+1]))
        return total_appeal
        
```

so python faster than rust. lets goo

okay i may have spoken too early. the leetcode just showed the last ran test case
so idk if the python one failed on the larger input and rust on smaller input or not

my bad
but i still think the python ones faster. lets see if i can do this in rust

```rust
use std::collections::HashSet;
impl Solution {
    pub fn appeal_sum(s: String) -> i64 {
        let s = s.as_bytes();
        let n = s.len();
        let mut total_appeal = n as i64;
        for size in 1..n + 1 {
            for i in 0..n - size {
                let s: HashSet<u8> = HashSet::from_iter(s[i..i+size+1].to_vec());
                total_appeal += s.len() as i64;
            }
        }
        total_appeal
    }
}
```

still didnt work tho

there has to be someway to do this in $n^2$ time.
if only i could find the appeal of the substring in constant time.
because i don't think i can do this without looping through each substring
or can i

```rust
use std::collections::HashSet;
impl Solution {
    pub fn appeal_sum(s: String) -> i64 {
        let s: Vec<u8> = s.chars().map(|elem| elem as u8 - 97).collect();
        let n = s.len();
        let mut total_appeal = n;
        for size in 1..n {
            let mut set: HashSet<u8> = HashSet::with_capacity(26);
            let mut counts: [usize; 26] = [0; 26];
            for j in 0..size {
                counts[s[j] as usize] += 1;
                set.insert(s[j]);

            }
            for i in size..n {
                set.insert(s[i]);
                counts[s[i] as usize] += 1;
                total_appeal += set.len();
                if counts[s[i-size] as usize] == 1 {
                    set.remove(&s[i-size]);
                }
                counts[s[i-size] as usize] -= 1;
            }
        }
        total_appeal as i64
    }
}
```

this one definitely passed the testcase the previous two didn't pass. but it was still not enough.
but why

is this really some divide and conquer bs that im not getting

im really lost with this one

i think im gonna dip.

i used flamegraph and i think it said 95% time is taken by the HashSet.insert() function call.
I'll try to thing of a way to not use hashset or

```rust
pub fn appeal_sum(s: String) -> i64 {
    let s: Vec<u8> = s.chars().map(|elem| elem as u8 - 97).collect();
    let n = s.len();
    let mut total_appeal = n;
    for size in 1..n {
        let mut set: HashSet<u8> = HashSet::with_capacity(26);
        let mut counts: [usize; 26] = [0; 26];
        for j in 0..size {
            let idx = s[j] as usize;
            counts[idx] += 1;
            if counts[idx] <= 1 {
                set.insert(s[j]);
            }
        }
        for i in size..n {
            let idx = s[i] as usize;
            counts[s[i] as usize] += 1;
            if counts[idx] <= 1 {
                set.insert(s[i]);
            }
            total_appeal += set.len();
            if counts[s[i - size] as usize] == 1 {
                set.remove(&s[i - size]);
            }
            counts[s[i - size] as usize] -= 1;
        }
    }
    total_appeal as i64
}
```

only insert if there isnt already stuff in it. which i feel like made it fast but not enough

I think this is it for today.
maybe ill continue this tomorrow or some other day
