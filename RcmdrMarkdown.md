<!-- R Commander Markdown Template -->

Homework #3
=======================

### Taylor Thul

### 2017-02-14






Get the dataset into R.


```r
> names(White_wines) <- make.names(names(White_wines))
```


```r
> library(abind, pos=18)
```



```r
> library(e1071, pos=19)
```

Run some summary statistics: 


```r
> numSummary(White_wines[,c("alcohol", "chlorides", "citric.acid", "density", 
+   "fixed.acidity", "free.sulfur.dioxide", "pH", "quality", "residual.sugar", 
+   "sulphates", "total.sulfur.dioxide", "volatile.acidity")], 
+   statistics=c("mean", "sd", "IQR", "quantiles"), quantiles=c(0,.25,.5,.75,1))
```

```
                             mean           sd        IQR      0%
alcohol               10.51426705  1.230620568  1.9000000 8.00000
chlorides              0.04577236  0.021847968  0.0140000 0.00900
citric.acid            0.33419151  0.121019804  0.1200000 0.00000
density                0.99402738  0.002990907  0.0043775 0.98711
fixed.acidity          6.85478767  0.843868228  1.0000000 3.80000
free.sulfur.dioxide   35.30808493 17.007137325 23.0000000 2.00000
pH                     3.18826664  0.151000600  0.1900000 2.72000
quality                5.87790935  0.885638575  1.0000000 3.00000
residual.sugar         6.39141486  5.072057784  8.2000000 0.60000
sulphates              0.48984688  0.114125834  0.1400000 0.22000
total.sulfur.dioxide 138.36065741 42.498064554 59.0000000 9.00000
volatile.acidity       0.27824112  0.100794548  0.1100000 0.08000
                             25%       50%      75%      100%    n
alcohol                9.5000000  10.40000  11.4000  14.20000 4898
chlorides              0.0360000   0.04300   0.0500   0.34600 4898
citric.acid            0.2700000   0.32000   0.3900   1.66000 4898
density                0.9917225   0.99374   0.9961   1.03898 4898
fixed.acidity          6.3000000   6.80000   7.3000  14.20000 4898
free.sulfur.dioxide   23.0000000  34.00000  46.0000 289.00000 4898
pH                     3.0900000   3.18000   3.2800   3.82000 4898
quality                5.0000000   6.00000   6.0000   9.00000 4898
residual.sugar         1.7000000   5.20000   9.9000  65.80000 4898
sulphates              0.4100000   0.47000   0.5500   1.08000 4898
total.sulfur.dioxide 108.0000000 134.00000 167.0000 440.00000 4898
volatile.acidity       0.2100000   0.26000   0.3200   1.10000 4898
```
Make sure no data is missing:


```r
> sapply(White_wines, function(x)(sum(is.na(x)))) # NA counts
```

```
       fixed.acidity     volatile.acidity          citric.acid 
                   0                    0                    0 
      residual.sugar            chlorides  free.sulfur.dioxide 
                   0                    0                    0 
total.sulfur.dioxide              density                   pH 
                   0                    0                    0 
           sulphates              alcohol              quality 
                   0                    0                    0 
     residual.sugar2                group 
                   0                    0 
```
No missing data! Hooray!
Look at the distribution of the variables:


```r
> #appears normal
> with(White_wines, Hist(quality, scale="frequency", breaks="Sturges", 
+   col="darkgray"))
```

<img src="figure/unnamed-chunk-19-1.png" title="plot of chunk unnamed-chunk-19" alt="plot of chunk unnamed-chunk-19" width="750" />

Scatterplot matric of alcohol, chlorides, citric acid and quality

```r
> scatterplotMatrix(~alcohol+chlorides+citric.acid+quality, reg.line=FALSE, 
+   smooth=FALSE, spread=FALSE, span=0.5, ellipse=FALSE, levels=c(.5, .9), 
+   id.n=0, diagonal = 'density', data=White_wines)
```

<img src="figure/unnamed-chunk-20-1.png" title="plot of chunk unnamed-chunk-20" alt="plot of chunk unnamed-chunk-20" width="750" />

Scatterplot of density, fixed acidity, free sulfur dioxide and pH


```r
> scatterplotMatrix(~density+fixed.acidity+free.sulfur.dioxide+pH, 
+   reg.line=FALSE, smooth=FALSE, spread=FALSE, span=0.5, ellipse=FALSE, 
+   levels=c(.5, .9), id.n=0, diagonal = 'density', data=White_wines)
```

<img src="figure/unnamed-chunk-21-1.png" title="plot of chunk unnamed-chunk-21" alt="plot of chunk unnamed-chunk-21" width="750" />

Scatterplot of residual sugar, sulphates, total sulfur dioxide and volatile acidity.  Residual sugar may need transformed. 


```r
> scatterplotMatrix(~residual.sugar+sulphates+total.sulfur.dioxide+volatile.acidity,
+    reg.line=FALSE, smooth=FALSE, spread=FALSE, span=0.5, ellipse=FALSE, 
+   levels=c(.5, .9), id.n=0, diagonal = 'density', data=White_wines)
```

<img src="figure/unnamed-chunk-22-1.png" title="plot of chunk unnamed-chunk-22" alt="plot of chunk unnamed-chunk-22" width="750" />
look at residual sugar on it's own


```r
> with(White_wines, Hist(residual.sugar, scale="frequency", breaks="Sturges", 
+   col="darkgray"))
```

<img src="figure/unnamed-chunk-23-1.png" title="plot of chunk unnamed-chunk-23" alt="plot of chunk unnamed-chunk-23" width="750" />
add new variable of log of sugar residuals

```r
> White_wines$residual.sugar2 <- with(White_wines, log2(residual.sugar))
```
it doesn't really look more normal 


```r
> with(White_wines, Hist(residual.sugar2, scale="frequency", breaks="Sturges",
+    col="darkgray"))
```

<img src="figure/unnamed-chunk-25-1.png" title="plot of chunk unnamed-chunk-25" alt="plot of chunk unnamed-chunk-25" width="750" />


```r
> library(nortest, pos=20)
```


```r
> with(White_wines, shapiro.test(residual.sugar))
```

```

	Shapiro-Wilk normality test

data:  residual.sugar
W = 0.88457, p-value < 2.2e-16
```
the Shapiro-Wilk is < 0.05 so we accept the hypothesis the data is not normally distibuted.
so try the Shapiro- Wilk on the transformed data 


```r
> with(White_wines, shapiro.test(residual.sugar2))
```

```

	Shapiro-Wilk normality test

data:  residual.sugar2
W = 0.9306, p-value < 2.2e-16
```
this did not help at all... still not normal.
moving on to the regression model


```r
> RegModel.1 <- 
+   lm(quality~alcohol+chlorides+citric.acid+density+fixed.acidity+free.sulfur.dioxide+pH+residual.sugar+sulphates+total.sulfur.dioxide+volatile.acidity,
+    data=White_wines)
> summary(RegModel.1)
```

```

Call:
lm(formula = quality ~ alcohol + chlorides + citric.acid + density + 
    fixed.acidity + free.sulfur.dioxide + pH + residual.sugar + 
    sulphates + total.sulfur.dioxide + volatile.acidity, data = White_wines)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.8348 -0.4934 -0.0379  0.4637  3.1143 

Coefficients:
                       Estimate Std. Error t value Pr(>|t|)    
(Intercept)           1.502e+02  1.880e+01   7.987 1.71e-15 ***
alcohol               1.935e-01  2.422e-02   7.988 1.70e-15 ***
chlorides            -2.473e-01  5.465e-01  -0.452  0.65097    
citric.acid           2.209e-02  9.577e-02   0.231  0.81759    
density              -1.503e+02  1.907e+01  -7.879 4.04e-15 ***
fixed.acidity         6.552e-02  2.087e-02   3.139  0.00171 ** 
free.sulfur.dioxide   3.733e-03  8.441e-04   4.422 9.99e-06 ***
pH                    6.863e-01  1.054e-01   6.513 8.10e-11 ***
residual.sugar        8.148e-02  7.527e-03  10.825  < 2e-16 ***
sulphates             6.315e-01  1.004e-01   6.291 3.44e-10 ***
total.sulfur.dioxide -2.857e-04  3.781e-04  -0.756  0.44979    
volatile.acidity     -1.863e+00  1.138e-01 -16.373  < 2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 0.7514 on 4886 degrees of freedom
Multiple R-squared:  0.2819,	Adjusted R-squared:  0.2803 
F-statistic: 174.3 on 11 and 4886 DF,  p-value: < 2.2e-16
```

whoops forgot to seperate into training and test data

```r
> set.seed(20170214)
> White_wines$group <- runif(length(White_wines$parent), min = 0, max = 1)
```

```
Error in `$<-.data.frame`(`*tmp*`, "group", value = numeric(0)): replacement has 0 rows, data has 4898
```

```r
> summary(White_wines)
```

```
 fixed.acidity    volatile.acidity  citric.acid     residual.sugar  
 Min.   : 3.800   Min.   :0.0800   Min.   :0.0000   Min.   : 0.600  
 1st Qu.: 6.300   1st Qu.:0.2100   1st Qu.:0.2700   1st Qu.: 1.700  
 Median : 6.800   Median :0.2600   Median :0.3200   Median : 5.200  
 Mean   : 6.855   Mean   :0.2782   Mean   :0.3342   Mean   : 6.391  
 3rd Qu.: 7.300   3rd Qu.:0.3200   3rd Qu.:0.3900   3rd Qu.: 9.900  
 Max.   :14.200   Max.   :1.1000   Max.   :1.6600   Max.   :65.800  
   chlorides       free.sulfur.dioxide total.sulfur.dioxide
 Min.   :0.00900   Min.   :  2.00      Min.   :  9.0       
 1st Qu.:0.03600   1st Qu.: 23.00      1st Qu.:108.0       
 Median :0.04300   Median : 34.00      Median :134.0       
 Mean   :0.04577   Mean   : 35.31      Mean   :138.4       
 3rd Qu.:0.05000   3rd Qu.: 46.00      3rd Qu.:167.0       
 Max.   :0.34600   Max.   :289.00      Max.   :440.0       
    density             pH          sulphates         alcohol     
 Min.   :0.9871   Min.   :2.720   Min.   :0.2200   Min.   : 8.00  
 1st Qu.:0.9917   1st Qu.:3.090   1st Qu.:0.4100   1st Qu.: 9.50  
 Median :0.9937   Median :3.180   Median :0.4700   Median :10.40  
 Mean   :0.9940   Mean   :3.188   Mean   :0.4898   Mean   :10.51  
 3rd Qu.:0.9961   3rd Qu.:3.280   3rd Qu.:0.5500   3rd Qu.:11.40  
 Max.   :1.0390   Max.   :3.820   Max.   :1.0800   Max.   :14.20  
    quality      residual.sugar2       group          
 Min.   :3.000   Min.   :-0.7370   Min.   :0.0002833  
 1st Qu.:5.000   1st Qu.: 0.7655   1st Qu.:0.2537537  
 Median :6.000   Median : 2.3785   Median :0.5034640  
 Mean   :5.878   Mean   : 2.1365   Mean   :0.5033795  
 3rd Qu.:6.000   3rd Qu.: 3.3074   3rd Qu.:0.7564856  
 Max.   :9.000   Max.   : 6.0400   Max.   :0.9993326  
```

```r
> #then it splits the data using the group, have to have plenty of data to do this and the split is 90/10 
> 
> #what random forests do is this process over and over again and makes the aggregate which might be called bootstrapping?
> 
> Galton.train <- subset(Galton, group <= 0.90)
```

```
Error in eval(expr, envir, enclos): object 'group' not found
```

```r
> Galton.test <- subset(Galton, group > 0.90)
```

```
Error in eval(expr, envir, enclos): object 'group' not found
```

