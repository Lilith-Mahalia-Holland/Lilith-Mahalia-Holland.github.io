---
title: "Module #5 assignment"
date: 2021-09-25
categories:
  - school
  - usf
  - lis-4273
tags:
  - assignment
  - statistics
  - fall-2021
  - questions-missing
excerpt: This section was over hypothesis testing and correlation.
---

## Question 1:
### A:
 - Null hypothesis HO = 70 and the alternative hypothesis HA != 70.

### B:
 - Meeting specifications?
   - TRUE 0.07186064 falls within the confidence interval of 0.05 and 0.95
   - The null hypothesis is true.

### C:
 - What is the p value? 
   - 0.07186064 
   - Since the P value is within the confidence intervals 0.05 and 0.95 the values shows that the mean is within reason for this test and passes the null hypothesis. It should be noted that the P value is close to the lower confidence interval and thus is a less likely result.

### D:
 - Meeting specifications? 
   - FALSE 0.0003182172 falls outside of the confidence interval of 0.05 and 0.95 
   - The mean is no longer within the confidence interval and thus fails the null hypothesis.

### E:
 - Meeting specifications? 
   - FALSE 0.04550026 falls outside of the confidence interval of 0.05 and 0.95 
   - The mean is no longer within the confidence interval and thus fails the null hypothesis.

## Question 2:
 - Using algebra I reworked the equation to be in terms of a and solved for the range of sample means. 
 - The lower range that the sample mean can be is: 85.06271 
   - Does this mean fall within the confidence interval? 
   - The value 0.95 falls within the 0.05 and 0.95 confidence interval as an extreme. 
   - TRUE
 - The upper range that the sample mean can be is: 86.95996
   - Does this mean fall within the confidence interval? 
   - The value 0.05 falls within the 0.05 and 0.95 confidence interval as an extreme.
   - TRUE
 
## Question 3:
### A:
 - The standard correlation coefficient is: 0.9999213

### B:
 - The Pearson correlation coefficient is: 0.9999213

### C:
{%
    include figure
    image_path="/assets/images/USF/LIS-4273/Module-5.png"
    alt="Module 6 Assignment Graphic, shows Male & Female student hours pearson correlation."
    caption="Figure 1: Pearson correlation visual."
%}

## R Script:

```r
cat(“\014”)

#Question 1:
cat(“Q1,A:\n”)
cat(“Null hypothesis H0 = 70 and the alternative hypothesis HA != 70.\n\n”)

cat(“Q1,B:\n”)
alpha <- 0.05
a <- 69.1
s <- 3.5
n <- 49
xbar <- 70
z <- (xbar-a)/(s/sqrt(n))
z <- 2*pnorm(-abs(z))
cat(“Meeting specifications?\n”)
cat(z > alpha & z < 1-alpha, z, “falls within the confidence interval of 0.05 and 0.95\n”)
cat(“The null hypothesis is true.\n\n”)

cat(“Q1,C:\n”)
cat(“What is the p value?\n”)
cat(z, “\n” )
cat(“Since the P value is within the confidence intervals 0.05 and 0.95 the values shows
that the mean is within reason for this test and passes the null hypothesis. It should be
noted that the P value is close to the lower confidence interval and thus is less likely result.\n\n”)

cat(“Q1,D:\n”)
alpha <- 0.05
a <- 69.1
s <- 1.75
n <- 49
xbar <- 70
z <- (xbar-a)/(s/sqrt(n))
z <- 2*pnorm(-abs(z))
cat(“Meeting specifications?\n”)
cat(z > alpha & z < 1-alpha, z, “falls outside of the confidence interval of 0.05 and 0.95\n”)
cat(“The mean is no longer within the confidence interval and thus fails the null hypothesis.\n\n”)

cat(“Q1,E:\n”)
alpha <- 0.05
a <- 69
s <- 3.5
n <- 49
xbar <- 70
z <- (xbar-a)/(s/sqrt(n))
z <- 2*pnorm(-abs(z))
cat(“Meeting specifications?\n”)
cat(z > alpha & z < 1-alpha, z,“falls outside of the confidence interval of 0.05 and 0.95\n”)
cat(“The mean is no longer within the confidence interval and thus fails the null hypothesis.\n\n\n”)

#Question 2:
cat(“Q2:\n”)
z <- qnorm(0.95/2)
s <- 8
n <- 64
xbar <- 85
a <- xbar - (z*(s/sqrt(n)))
z <- (xbar-a)/(s/sqrt(n))
z <- 2*pnorm(-abs(z))
cat(“Using algebra I reworked the equation to be in terms of a and solved for the range of sample means.\n”)
cat(“The lower range that the sample mean can be is:”, a, “\n”)
cat(“Does this mean fall within the confidence interval?\n”)
cat(“The value”, z, “falls within the 0.05 and 0.95 confidence interval as an extreme.\n”)
cat(z > 0.05 & 0.95 > z, “\n”)

z <- qnorm(0.05/2)
s <- 8
n <- 64
xbar <- 85
a <- xbar - (z*(s/sqrt(n)))
z <- (xbar-a)/(s/sqrt(n))
z <- 2*pnorm(-abs(z))
cat(“The upper range that the sample mean can be is:”, a, “\n”)
cat(“Does this mean fall within the confidence interval?\n”)
cat(“The value”, z, “falls within the 0.05 and 0.95 confidence interval as an extreme.\n”)
cat(z > 0.05 & 0.95 > z, “\n\n\n”)

#Question 3
cat(“Q3:\n”)
x <- c(4,5,6,19,22,28)
y <- c(4,5,6,18.9,22.2,27.8)
df <- data.frame(x,y)
cNorm <- cor(df)
cat(“The standard correlation coefficient is:”, cNorm[2,1], “\n”)

cPear <- cor(df, method = “pearson”)
cat(“The pearson correlation coefficient is:”, cPear[2,1], “\n”)

corrgram(df)
title(“Male & Female student hours correlation”)
```