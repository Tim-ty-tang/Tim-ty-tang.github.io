---
title: "Distances and Fuzzy Matching"
date: "2025-02-02"
summary: "learning fuzzy matching"
description: "learning fuzzy matching"
toc: false
readTime: true
---
## Distances and Fuzzy Matching

### Fuzzy Matching

#### Levenshtein

Levenshtein distance is effectively the edit distance of two strings, by definition, it is the min number of single character edits to get to the change. Edits can be substitution, insertion or deletions:

Examples:

```html
Let's calculate the Levenshtein distance between "kitten" and "sitting":

1. kitten → sitten (substitution of "s" for "k")
2. sitten → sittin (substitution of "i" for "e")
3. sittin → sitting (insertion of "g" at the end)

Levenshtein distance = 3
```

So for fuzzy matching, generally this would be what we are looking for: given two str vectors how effectively far they are.

Naturally this does it give any weightings to the the mistake made, so for example, in Levenshtein, you'd have a larger distance metric for swapped letters than missing letters in a mistyped word. That might not be desirable.

Algorithm wise, here is the standard pseudo-code for Wagner-Fischer algorithm to implement, this is a canonical example of dynamic programming:

```html
function LevenshteinDistance(char s[1..m], char t[1..n]):
  // for all i and j, d[i,j] will hold the Levenshtein distance between
  // the first i characters of s and the first j characters of t
  declare int d[0..m, 0..n]
 
  set each element in d to zero
 
  // source prefixes can be transformed into empty string by
  // dropping all characters
  for i from 1 to m:
    d[i, 0] := i
 
  // target prefixes can be reached from empty source prefix
  // by inserting every character
  for j from 1 to n:
    d[0, j] := j
 
  for j from 1 to n:
    for i from 1 to m:
      if s[i] = t[j]:
        substitutionCost := 0
      else:
        substitutionCost := 1

      d[i, j] := minimum(d[i-1, j] + 1,                   // deletion
                         d[i, j-1] + 1,                   // insertion
                         d[i-1, j-1] + substitutionCost)  // substitution
 
  return d[m, n]
```

#### Hamming

Hamming distance is edit distance with only subs, so in this case for hamming distance to work, the strings need to be confirmed the same.

The implementation is the same, except of d[i, j] for obvious comparisons.

This is more restrictive in fuzzy search applications but for error detection and error corrections it is much more relevant.

#### Longest Common Subsequence

Longest common sequence is just finding the longest substring that both strings share.

Again this is a canonical dynamic programming problem, as a substring of a substring is also a common sequence. It is the basis for the Wagner-Fischer Algo.

For LCS length, LCS reads, and diff reads refer to wikipedia, which has pseudo code implementations.

#### Damerau-Levenshtein

Dameru-Levenshtein is Levenshtein but includes transposition of adjacent characters as an allowed action. This reduces the effective distance if words are jumbled up instead of straight-up wrong. Adding transposition is a complexity add, so in practise we have optimal string alignment distance or restricted edit distance.

In the case of optimal string alignment, the condition is the number of edits ops needed to make a string equal with no substring being edited more than once.

Using **CA** and **ABC** as an example: LD(CA, ABC) = 2, but OSA(CA, ABC) = 3

**CA** → **AC** → **ABC** the same substring is edited twice.
**CA** → **A** → **AB** → **ABC**

Here is the peudo code for Optimal String Alignement distance:

```html
**algorithm** OSA-distance **is**
    **input**: strings a[1..length(a)], b[1..length(b)]
    **output**: distance, integer
    
    **let** d[0..length(a), 0..length(b)] be a 2-d array of integers, dimensions length(a)+1, length(b)+1
    _// note that d is zero-indexed, while a and b are one-indexed._
    
    **for** i := 0 **to** length(a) **inclusive** **do**
        d[i, 0] := i
    **for** j := 0 **to** length(b) **inclusive** **do**
        d[0, j] := j
    
    **for** i := 1 **to** length(a) **inclusive** **do**
        **for** j := 1 **to** length(b) **inclusive** **do**
            **if** a[i] = b[j] **then**
                cost := 0
            **else**
                cost := 1
            d[i, j] := minimum(d[i-1, j] + 1,     _// deletion_
                               d[i, j-1] + 1,     _// insertion_
                               d[i-1, j-1] + cost)  _// substitution_
            **if** i > 1 and j > 1 and a[i] = b[j-1] and a[i-1] = b[j] **then**
                d[i, j] := minimum(d[i, j],
                                   d[i-2, j-2] + 1)  _// transposition_
    **return** d[length(a), length(b)]
```

A real DL-distance places no substring restrictions.

Here is the pseudo code with adjacent transpositions:

```html
**algorithm** DL-distance **is**
    **input**: strings a[1..length(a)], b[1..length(b)]
    **output**: distance, integer
    
    da := new array of |Σ| integers
    **for** i := 1 **to** |Σ| **inclusive** **do**
        da[i] := 0
    
    **let** d[−1..length(a), −1..length(b)] be a 2-d array of integers, dimensions length(a)+2, length(b)+2
    _// note that d has indices starting at −1, while a, b and da are one-indexed._
    
    maxdist := length(a) + length(b)
    d[−1, −1] := maxdist
    **for** i := 0 **to** length(a) **inclusive** **do**
        d[i, −1] := maxdist
        d[i, 0] := i
    **for** j := 0 **to** length(b) **inclusive** **do**
        d[−1, j] := maxdist
        d[0, j] := j
    
    **for** i := 1 **to** length(a) **inclusive** **do**
        db := 0
        **for** j := 1 **to** length(b) **inclusive** **do**
            k := da[b[j]]
            ℓ := db
            **if** a[i] = b[j] **then**
                cost := 0
                db := j
            **else**
                cost := 1
            d[i, j] := minimum(d[i−1, j−1] + cost,  _//substitution_
                               d[i,   j−1] + 1,     _//insertion_
                               d[i−1, j  ] + 1,     _//deletion_
                               d[k−1, ℓ−1] + (i−k−1) + 1 + (j-ℓ−1)) _//transposition_
        da[a[i]] := i
    **return** d[length(a), length(b)]
```

#### Jaro-Winkler

A variant of Jaro distance, the Jaro-Winkler uses a prefix. Jaro distance is the transposition distance between 2 strings. and Jaro-Winkler takes the prefix scale as a bias.

Good to know, but not terribly relevant.

#### Cosine Similarity

Cosine Similarity is a vector check metric by taking the dot product of 2 vectors dividing by the magnitude.

This is very common in indexed docs or indexed strings.

The cosine similarity is calculated using the following formula:

cosine similarity = (vector1 x vector2) / (|vector1| x |vector2|)

where:

* vector1 and vector2 are the two vectors being compared
* |vector1| and |vector2| are the magnitudes of the two vectors

**Example**: Let's calculate the cosine similarity between two documents, represented by the following vectors:

vector1 = [1, 2, 3, 4, 5]
vector2 = [2, 4, 6, 8, 10]

The dot product of the two vectors is:

vector1 x vector2 = 1 x 2 + 2 x 4 + 3 x 6 + 4 x 8 + 5 x 10 = 100

The magnitudes of the two vectors are:

|vector1| = sqrt(1^2 + 2^2 + 3^2 + 4^2 + 5^2) = sqrt(55)
|vector2| = sqrt(2^2 + 4^2 + 6^2 + 8^2 + 10^2) = sqrt(220)

The cosine similarity is therefore:

cosine similarity = 100 / (sqrt(55) x sqrt(220)) = 0.816

#### Libraries

Use python Fuzzy-wuzzy (defaults LD), or rapidfuzz for perf.
