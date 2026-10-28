---
title: "Module #6 assignment"
date: 2021-10-02
categories:
  - school
  - usf
  - lis-4273
tags:
  - assignment
  - statistics
  - fall-2021
  - questions-missing
excerpt: Population Standard Deviation assignment
---
## Problem A:
### A1:
 - The population mean is 11.8

### A2:
 - The random sample for this execution is 11 and 8


{%
include figure
image_path="/assets/images/USF/LIS-4273/Module-6.png"
alt="Module 6 Assignment Graphic, shows the statistic results of the Population Standard Deviation (PSD) function."
caption="Figure 1: Population Standard Deviation results."
%}
### A3:
 - The mean of the sample is 9.5 while the standard deviation of the sample is 2.12

### A4:
 - The sample mean is 9.5 while the population mean is 11.8, a difference of 2.2. The sample standard deviation is 2.12 while the population standard deviation is 2.86, a difference of .74.

## Problem B:
### B1:
 - Since n = 100, p = 0.95, and q = 1-p = 0.05 when np and nq are >= 5 the sample is approximately normal, in this case np = 95 and nq = 5. Since both are true the sample is approximately normal.

### B2:
 - For the sampling distribution to be approximately normal np and nq must be >= 5, using algebra it can be determined that the minimum number of samples is just n >= 5/p or n >= 5/q. In this case for p it would be 5.263158 and for q it would be 100 since we only care for p, with rounding 6 samples are required.

## Problem C:
 - The mean of simulating a coin flip with the sample function is 0.499702 While the mean of simulating a coin flip with the rbinom function is 0.5004 both of these means are within reason when you consider the number of flips being 1e+06 . The main difference in these functions is the ease of use of the rbinom function requiring no extra effort while the sample function requires the creation of a loop. The speed at which the two functions run at are dramatically different, at 1e+06 flips it takes the sample loop 4.31 seconds to execute while the binom flips take 0.06 seconds to execute.


## R Script:

```r
cat(“\014”)

psd <- function(x){
 sqrt((length(x)-1)/length(x)) * sd(x)
}

#A
populationA <- c(8,14,16,10,11)
#A,A
populationA_Mean <- mean(populationA)
populationA_SD <- psd(populationA)
#A,B
sampleA <- sample(populationA, 2)

sampleA_Mean <- mean(sampleA)
sampleA_SD <- sd(sampleA)
#A,C
cat(“\nThe population mean is”, populationA_Mean, “while the sample mean is”, sampleA_Mean,“\n”)
#A,D
cat(“\nThe population standard deviation is”, populationA_SD, “while the sample standard deviation is”, sampleA_SD, “\n”)

#B
proportionB <- 0.95
#1
cat(“\n\nnp =”, proportionB*100,“nq =”, (1-proportionB)*100,“both are >= 5\n this means that the sample is approximately normal.\n”)
#2
cat(“\nFor the sampling distribution to be approximately normal np and nq must be >= 5, using algebra
   it can be determined that the minimum number of samples is just n >= 5/p or n >= 5/q. In this case for
   p it would be”, 5/proportionB, “and for q it would be”, 5/(1-proportionB), “since we only care for p,
   with rounding”,ceiling(5/proportionB), “samples are required.\n”)

#C
flipN <- 1000000
sampleFlips <- NULL
ptm <- proc.time()
for (i in 1:flipN){
 sampleFlips[i] <- sample(c(0,1),1)
}
sampleTime <- proc.time() - ptm

ptm <- proc.time()
binomFlips <- rbinom(flipN, 1, 0.5)
binomTime <- proc.time() - ptm

cat(“\nThe mean of simulating a coin flip with the sample function is”, mean(sampleFlips), “While the mean of
   simulating a coin flip with the rbinom function is”, mean(binomFlips), “both of these means are within reason
   when you consider the number of flips being”, flipN, “. The main difference in these functions is the ease of use
   of the rbinom function requiring no extra effort while the sample function requires the creation of a loop.
   The speed at which the two functions run at are dramatically different, at”, flipN, “flips it takes the sample
   loop”, unname(sampleTime[3]), “seconds to execute while the binom flips take”, unname(binomTime[3]), “seconds to execute.”)
```