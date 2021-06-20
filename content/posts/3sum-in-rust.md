---
title: "3SUM in Rust"
description: "Solving the famous 3SUM problem in Rust"
date: 2021-06-10T21:44:00Z
draft: false
---

# 3SUM in Rust

3SUM is a famous problem in computer science, and often comes up in Software Engineering interview questions of the leetcode variety. Here we will present a simple solution in Rust.

There are variations to how the problem is generally formulated. The formulation we will discuss is:

* Given an array of integers, find all triplets that sum to zero, i.e. `a + b + c = 0`
* Do not output duplicates.

Example:

* Input: `[-4, -6, 2, 2, 0, 1, 3, -2]`
* Output: `[[-4, 1, 3], [-4, 2, 2], [-2, 0, 2]]`

There are multiple possible solutions to the problem. We could consider a brute-force solution and the merits of that over a more optimised solution, but in the interest of time, the solution presented here is O(n²).

We start out by sorting the array of values passed to the function. This is an O(n log n) operation. We then traverse the sorted array, and for each element in the array, we consider three indices: *start*, *left* and *right*.

The *start* index points to the element we are currently looking at while traversing the array. The *left* index points to the next smallest element. The *right* index points to the largest element.

We then either increment *left* or decrement *right*, depending on how close the total sum of the elements pointed to by our indices is to 0.

If the sum is more than 0, we decrement *right* and by doing so decrease the whole sum. If the sum is less than 0, we increment *left* and increase the whole sum.

If the sum is equal to 0, we add the triplet to the output of the function.

As the problem states we should not output duplicates, we skip over the duplicate elements while traversing the array and incrementing/decrementing the *left* and *right* indices.

{{< highlight rust >}}
pub fn three_sum(values: &Vec<i32>) -> Vec<Vec<i32>> {
    let mut sorted = values.clone();
    sorted.sort();
    let mut ret = Vec::new();
    for (start, elem) in sorted.iter().enumerate() {
        if start > 0 && sorted[start-1] == *elem {
            // if this element is the same as the previous one, we don't do
            // anything, this is to avoid outputting duplicates
            continue;
        }
        let mut left = start+1;
        let mut right = sorted.len()-1;
        while left < right {
            if sorted[left] + sorted[right] == -elem {
                ret.push(vec![*elem, sorted[left], sorted[right]]);
                left = align_left(&sorted, left);
                right = align_right(&sorted, right, start);
            } else if sorted[left] + sorted[right] > -elem {
                right = align_right(&sorted, right, start);
            } else {
                left = align_left(&sorted, left);
            }
        }
    }
    ret
}

fn align_left(values: &Vec<i32>, left: usize) -> usize {
    let mut ret = left + 1;
    while ret < values.len() && values[ret] == values[ret-1] {
        // to avoid outputting duplicates we keep incrementing our index
        // if we encounter the same element
        ret += 1;
    }
    ret
}

fn align_right(values: &Vec<i32>, right: usize, start: usize) -> usize {
    let mut ret = right - 1;
    while ret > start && values[ret] == values[ret+1] {
        // to avoid outputting duplicates we keep decrementing our index
        // if we encounter the same element
        ret -= 1;
    }
    ret
}
{{< /highlight >}}
