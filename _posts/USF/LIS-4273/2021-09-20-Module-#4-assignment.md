---
title: "Module #4 assignment"
date: 2021-09-20
categories:
  - school
  - usf
  - lis-4273
tags:
  - assignment
  - statistics
  - fall-2021
  - questions-missing
excerpt: Standard probability assignment.
---
# Probabilities
## Problem A:
### A1:
 - A = 30, n = 90, P(A) = 1/3

### A2:
 - B = 30, n = 90, P(B) = 1/3

### A3:
 - A = 30, B = 30, n = 90, A and B = 10
 - P(A) = 1/3, P(B) = 1/3, P(A and B) = P(A) * P(B) = 1/9
 - P(A or B) = P(A) + P(B) - P(A and B) = 1/3 + 1/3 - 1/9 = 5/9

### A4:
 - A = 30, B = 30, n = 90
 - P(A or B) = P(A) + P(B) = 2/3

## Problem B:
### B1:
 - TRUE

### B2:
 - The mathematics follow the formula and are done correctly as well as the fact that the Bayes’ Theorem considers the accuracy of a prediction, unlike a classical probability scenario there is a potential for both a false negative and a false positive where the Bayes’ Theorem is applied. Since the accuracy is considered in the prediction the overall probability of an event isn’t determined by independent parts. It should also be stated that the event of rain is only a part of the sample space.

## Problem C:
```r
> dbinom(0, size = 10, prob = 0.2)
[1] 0.1073742
```

or

```r
> dbinom(10, size = 10, prob = 0.8)
[1] 0.1073742
```
