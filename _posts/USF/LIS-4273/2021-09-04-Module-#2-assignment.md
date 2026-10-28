---
title: "Module #2 assignment"
date: 2021-09-04
categories:
  - school
  - usf
  - lis4273
tags:
  - assignment
  - statistics
  - fall-2021
  - questions-missing
excerpt: A starter assignment for calculating the mean of a vector in R.
---

{% 
    include figure 
    image_path="/assets/images/USF/LIS-4273/Module-2.png"
    alt="Module 2 Assignment Graphic, shows the full RStudio layout with script." 
    caption="Figure 1: Full RStudio IDE script." 
%}

## R Script:

```r
> assignment2<- c(6,18,14,22,27,17,22,20,22)
> myMean <- function(x) {return(sum(x)/length(x))}
> myMean(assignment2)
[1] 18.66667
> myMean(1:5)
[1] 3
```

## Response:
For the step by step of the code I started with creating a vector called assignment2 that was filled with the required
values. I then created a function called myMean that is intended to take some input (be it a single value, vector, 
matrix, or any other compatible format) and take the sum of all the values in said input, the function will then take 
the total number of values in that input divide the sum by it and then return the result.

In this case the input was assignment2 which is a vector containing 9 values. the sum of said vector is 168 and the 
element count is 9. by dividing 168 by 9 we get the resulting mean of 18.66667. I further proved this by taking the 
mean of 1 to 5 which resulted in 3. In the provided picture I showed the internal of myMean being assignment2 instead 
it being x like in the text script. I simply did that for formatting reasons and it is functionally identical, it does 
not matter what is used inside the function as long as it is consisting within.