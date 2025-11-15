+++
title = "A problem a day: Day 4"
date = 2025-02-24
[taxonomies]
tags = ["algorithm", "a-problem-a-day"]
+++


## Origin

url: [Leetcode problem 48](https://leetcode.com/problems/rotate-image/description/)

### Problem

You are given an `n x n` 2D `matrix` representing an image, rotate the image by **90** degrees (clockwise).

You have to rotate the image *in-place*, which means you have to modify the input 2D matrix directly. **DO NOT** allocate another 2D matrix and do the rotation.

### Examples

#### Example 1

```c
[                                    [
[1, 2, 3],        --->               [7, 4, 1],
[4, 5, 6],        --->               [8, 5, 2],
[7, 8, 9]         --->               [9, 6, 3],
]                                    ]
```

**Input:** matrix = [ [1,2,3],[4,5,6],[7,8,9] ]
**Output:** [ [7,4,1],[8,5,2],[9,6,3] ]

#### Example 2

```c
Original Matrix:               Rotated Matrix (90 degrees clockwise):
|  5 |  1 |  9 | 11 |          | 15 | 13 |  2 |  5 |
|  2 |  4 |  8 | 10 |          | 14 |  3 |  4 |  1 |
| 13 |  3 |  6 |  7 |          | 12 |  6 |  8 |  9 |
| 15 | 14 | 12 | 16 |          | 16 |  7 | 10 | 11 |

```

### Constraints

- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 1000`

## Journey

In place rotation.
Is there a trap here somewhere or is it the issue of time.
cause I feel like you just reverse the rows and then transpose the resulting matrix and you get the 90 degree rotation

or does this not count as in-place. im not creating another 2d matrix so it must count right.

well it did work.

```rust
impl Solution {
    pub fn rotate(matrix: &mut Vec<Vec<i32>>) {
        let n = matrix.len();
        let mut temp: i32 = 0;
        for i in 0..n/2 {
            for j in 0..n {
                temp = matrix[i][j];
                matrix[i][j] = matrix[n-1-i][j];
                matrix[n-1-i][j] = temp;
            }
        }
        for i in 0..n {
            for j in i..n {
                temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
    }
}
```

## Destination

Well this is a first.
When I chose this question, I didn't really think much. I just saw matrix and inplace and thought to myself. well this should be interesting. Well it was interesting for a minute.

I don't know who labelled this problem as medium. The [A Problem a Day 2](./aproblemaday2.md) problem was medium and in hindsight It was not that difficult but it did require me to think about it the problem for a bit. I'm still doubting if I did it the right way or not.

Anyway. enough about this shitass of a problem.
how's life?

have you heard about mersenne prime. Now they very interesting. I might have a post coming up about it (spoilers).

so what do i add. It hasn't even crossed the 500 words. Its currenly at 407 words. wait no 408. wow that jumped to 411. 416, 417, 418, 419, 421. aha finally caught it.
dammit it jumped to 425 again. yes this is whats happening today.
don't blame me. blame the question. [*seriously questioning what am i doing?*]

wowowowoooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo

did you know I stopped reading the [Ffmpeg school of assembly](https://github.com/FFmpeg/asm-lessons) to do this question. I was like okay there are only three lessons and I have completed the first one so let me stop and do the a problem a day thing.

two words.
 ![word count of this note in obsidian](/images/apad4-words.png)
bye.
