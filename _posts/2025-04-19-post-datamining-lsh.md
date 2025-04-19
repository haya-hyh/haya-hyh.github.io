---
title: "【Data Mining 1】Locality-Sensitive Hashing(LSH) in Data Mining"
date: 2025-04-15
last_modified_at: 2025-04-15
categories:
  - Blog
  - Algorithm
tags:
  - LSH
  - Python
  - Data Mining
toc: true
layout: single
mathjax: true

---
An introduction to Locality-Sensitive Hashing (LSH), covering its core idea, how it works and a Python implementation.

---
## ✅Getting Ready

Before diving into the core concept of LSH, it's helpful to understand **Jaccard similarity**, since many problems can be framed as finding similar sets.

Jaccard similarity measures how similar two sets are by comparing their intersection and union.

No more talking — let's drop the formula:

\[
J(A, B) = \frac{|A \cap B|}{|A \cup B|}
\]

Where:
- \( A \) and \( B \) are two sets,
- \( |A \cap B| \) is the number of elements in common,
- \( |A \cup B| \) is the total number of unique elements in both sets.


## ✨How it works?
In real-world applications, comparing a large number of containers to find similar item sets using Jaccard similarity can be computationally and memory intensive. Locality-Sensitive Hashing (LSH) provides an efficient way to reduce the memory footprint by hashing similar items into the same buckets, enabling faster similarity detection.

The following example demonstrates how to use LSH to find similar documents. It includes some preprocessing steps such as shingling. If you're only interested in the details of LSH itself, feel free to jump to Step 3.
### Step 1: Shingling

In the first step, we convert each document into a set of shingles. A *k*-shingle is defined as a sequence of *k* tokens, where each token can be either a character or a word. When handling whitespace, it can be treated as a separate token or ignored, depending on the specific application.

The value of *k* is usually set to 5 for short documents, such as emails, while for longer documents, a larger *k* (e.g., 9 or 10) may be more appropriate.

For example, if we have a document `"abcde"`, we can represent it as a set of 3-shingles: `{"abc", "bcd", "cde"}`.

In general, a pair of documents represented as sets of *k*-shingles are considered more similar if they have a high Jaccard similarity.

### Step 2: Minhash
However, comparing all document pairs is still computationally expensive when dealing with a large number of documents.

As a first idea, we might consider building a characteristic matrix, where each **row** represents a *k*-shingle, and each **column** corresponds to a document. If a document contains a particular shingle, we mark the entry as `1`; otherwise, it's `0`.

This binary matrix representation looks more structured, and we can leverage many efficient libraries, such as NumPy, to perform matrix operations on it.

#### Example: Characteristic Matrix

| Shingle | Doc1 | Doc2 | Doc3 |
|---------|------|------|------|
| abc     | 1    | 1    | 0    |
| bcd     | 1    | 0    | 1    |
| cde     | 0    | 1    | 1    |
| def     | 1    | 1    | 0    |
| efg     | 0    | 0    | 1    |

Now, if we hash each row and look at the minimum hash value per document, the probability that two documents have the same minimum value exactly equals their Jaccard similarity — since only shared shingles can produce the same hash result.

In the matrix, we can simulate a hash function using a random **permutation of rows**. In this case, the minimum value corresponds to the **first row (shingle)** in the permutation where the document has a `1`.

By applying multiple independent permutations, we can estimate the Jaccard similarity based on how often the minimum values match.

But what If the number of rows is too large? Storing full permutations becomes infeasible. Still, we use hash functions to simulate permutations.

Each hash function maps a row index  to a pseudo-random value. The smaller the value, the earlier it appears in the simulated permutation. For each document and each pseudo-permutation we just keep the minimum hash value(**Minvalue**) among the rows where the document has a 1.
```text
for each row r do
    for each hash function hᵢ do
        compute hᵢ(r);
    for each column c
        if c has 1 in row r
            for each hash function hᵢ do
                if hᵢ(r) is smaller than M(i, c) then
                    M(i, c) := hᵢ(r);
```



And for each column (document), this sequence of **Minvalue** is called its signature.

### Step 3: LSH

![The big picture](/assets/img/lsh-diagram.png)
## 💻 Code implement
Will update here https://github.com/haya-hyh/datamining/tree/main/HW1
