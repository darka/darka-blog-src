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


The solution presented here runs in O(n²) time.

{{< highlight rust >}}
pub fn three_sum(values: &Vec<i32>) -> Vec<Vec<i32>> {
    let mut sorted = values.clone();
    sorted.sort();
    let mut ret = Vec::new();
    for (start, elem) in sorted.iter().enumerate() {
        if start > 0 && sorted[start-1] == *elem {
            // if this element is the same as the previous one, we don't do anything
            // this is to avoid outputting duplicates
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

fn align_left(sorted_values: &Vec<i32>, left: usize) -> usize {
    let mut ret = left + 1;
    while ret < sorted_values.len() && sorted_values[ret] == sorted_values[ret-1] {
        // to avoid outputting duplicates we keep incrementing our counter
        // if we encounter the same element
        ret += 1;
    }
    ret
}

fn align_right(sorted_values: &Vec<i32>, right: usize, start: usize) -> usize {
    let mut ret = right - 1;
    while ret > start && sorted_values[ret] == sorted_values[ret+1] {
        // to avoid outputting duplicates we keep decrementing our counter
        // if we encounter the same element
        ret -= 1;
    }
    ret
}
{{< /highlight >}}
