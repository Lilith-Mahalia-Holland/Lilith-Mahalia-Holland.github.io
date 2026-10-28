---
title: "Module #3 assignment"
date: 2021-09-08
categories:
  - school
  - usf
  - lis-4273
tags:
  - assignment
  - statistics
  - fall-2021
  - questions-missing
excerpt: A simple summary statistics assignment.
---

{% 
    include figure 
    image_path="/assets/images/USF/LIS-4273/Module-3-1.png"
    alt="Module 3 Assignment Graphic, shows the first half of the RStudio layout with script." 
    caption="Figure 1: RStudio IDE script part 1." 
%}
{%
    include figure
    image_path="/assets/images/USF/LIS-4273/Module-3-2.png"
    alt="Module 3 Assignment Graphic, shows the second half of the RStudio layout with script."
    caption="Figure 2: RStudio IDE script part 2."
%}

## Initial Values:
 - **Set A:**  10, 2, 3, 2, 4, 2, 5
 - **Set B:**  20, 12, 13, 12, 14, 12, 15

## Assignment Summary:

```r
Assignment summary:
                                 SetA                SetB
Mean:                               4                  14
Median:                             3                  13
Mode:                               2                  12
Range:                              8                   8
Min:                                2                  12
Max:                               10                  20

Interquartile:                      2.0,               12.0
                                    2.0,               12.0
                                    3.0,               13.0
                                    4.5,               14.5
                                   10.0                20.0

Variance:                           
                                8.333333            8.333333

Standard_deviation:
                                2.886751            2.886751

A summary of setA:
  Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
   2.0     2.0     3.0     4.0     4.5    10.0

A summary of setB:
  Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  12.0    12.0    13.0    14.0    14.5    20.0
```

## Response:
Both setA and setB have the same range, variance, and standard deviation. This is caused by the fact that setB is just 
setA multiplied by 10, and since all values are identical outside of that, equations that are not dependent on the 
location of the numbers relative to zero provide the same results.

However, for the mean, median, mode, min, max, and interquartile, all of the values are dependent on their location. 
For these values setB is just again 10x setA, this essentially just shows that setB is a scaled up version of setA and 
the only values impacted by this are the ones that are location depend.
