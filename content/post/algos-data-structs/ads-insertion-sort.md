---
author: "Giuseppe De Palma"
title: "Insertion sort it again."
description: "The first post where I try to implement algos and data structures, an excuse to go over them for myself."
categories: ["Algorithms & Data Structures"]
date: 2020-12-04T11:55:21+00:00
draft: false
tags: ["algorithms", "computer science"]
imagelink: "ADS/ADS1.png"
---

Often times I found in the internet computer science challenges as in "X days of algorithms" where people have to write each day about some algorithm. So I thought: I wanna do it too!

After some thought and knowing that I am a terribly slow writer, I decided to not take it so seriously and not do it every single day. I won't set a number of days and do it every day, I'd rather have it as an undefined amount of time where I can continue forever with this "challenge," but do it with my own time.

Yeah basically it's not a challenge at all I just want to go over algos and data structures I studied in university years ago, to brush them up and study others for the first time.

# Let's start with a simple one: Insertion Sort

Of course the sorting algorithms are an evergreen class of algos to start from. So let's take the good old Insertion Sort.

### The Algorithm

The idea behind this one is not difficult to grasp. With a sequence of numbers that needs sorting, we can take one element and "_insert_" it in his correct position, removing it from where it was.

Starting from the beginning of the sequence, we can consider the first number as **already sorted** and leave it where it is. Then going on the second number we compare it with the first. If it's greater, then it's all good as well, otherwise if smaller we take it and insert it behind the first number. We do this by shifting the previous number one position forward and putting the number we're sorting in the empty space at the start.

Now we have **2 sorted numbers**. Great.

We can go on on the third number, if it's greater than the one in the second position it's alright. We know for sure it is greater than the first number as well. Otherwise we have to start comparing.

We compare it with the previous' previous number, the first. If it is greater than then first number we have to insert it after, in between the first and second. If it's less than the first, then it becomes the first. We proceed like this for all the numbers, everytime we get a smaller than previous number we start checking backwards to see where to place it.

Honestly, rather than inserting a smaller number in another place in the sequence, we are **right** shifting all the previous bigger numbers, leaving an empty space to be filled. Seeing like this is easier when implementing Insertion Sort.

A gif from Wikipedia shows very well the procedure:

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/0/0f/Insertion-sort-example-300px.gif"
     alt="Insertion Sort gif. If you are reading this then the gif fucked up." />
</p>

### Complexity

Insertion Sort scans through a sequence analyzing every number. If the number of elements is **n**,
the algorithm will surely perfom **n** operations. For each number it goes back to check the previous numbers and move right the elements.

The worst case scenario is when the sequence is already ordered... **backwards**. The biggest number is at the start. For every single number we find ourselves in the case where we need to insert it into another place. So for every element we have to scan through the entire subsequence til the beginning. In computer science we like to keep things easy and clean so we can just say that the number of operations is: O(n<sup>2</sup>), without having to calculate the exact quantities.

An interesting fact is about the best case scenario, when the sequence is already sorted. Insertion sort is a O(n) algorithm in such cases. And when the sequences are extremely small it is extremely fast, more than the usual Quick/Merge sort. In many languages the default sorting algorithm is a combination of Quicksort and Insertion Sort, where it uses Insertion Sort when the sequence is very small (around 10 elements max if I'm not wrong, I guess they found out this number empirically).

### In Python

I wrote a simple python version just to have fun.

```python
def insertion_sort(seq):                    # A seq with length of n.
    for i in range(len(seq)):               # For from 0 to n-1.
    j = i - 1                                # In   dex for the previous number
        elem_to_sort = seq[i]
        while j >= 0 and seq[j] > elem_to_sort: # Til j is valid and the elem_to_sort is smaller
            seq[j+1] = seq[j]                      # we move right the others
            j -= 1
        seq[j+1] = elem_to_sort   # and we place elem_to_sort in the right spot
```
