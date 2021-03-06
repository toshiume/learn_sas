Inference statistics (one sample)
=================================

These are two ways to infer populatoin parameter by sammple.
1. Confidence Interval
2. Hypothesis test (p value)


Before Calculation : Interpretation of Confidence Interval
----------------------------------------------------------

Confidence interval is "an inference for population parameter".

Suppose the point estimate of population mean is 85 and the 95% confidence interval is (80, 90), what does it mean?

* False : If repeatedly sample observations and calculate their means, they lie within (80, 90) at the probability of 95%.
  + Because the CI is an estimate for population mean, not for sample mean. 
* False : There is a 95% chance that the ture mean lies between the confidence interval (80 90).
  + Because the ture mean is already set. Whether it lies within 80 to 90 is 0% or 100%.
* False : 95% of all observations' means lie in (80, 90)
  + Not about each observation but mean. Moreover, not for sample but for population.
* True : I have 95% confidence that the population mean lies within (80, 90).
* True : If we take 100 samples, 95 times we can get a confidence interval which covers population mean. However, this (80, 90) may not be the one, or (80 90) may not cover the population mean. If I can choose one from 100 CIs, the possiblity to cover the true(population ) mean is 95%, but where (80, 90) CI covers the true mean is 0% or 100%.



Confidence Interval of population mean (one variable)
-----------------------------------------------------

Sample mean distributoin can be described under specific assumptoin as follow,

(Assumption)
+ Simple random sample
+ Normal population or large sample(>30)
(Distribution)
* sample mean ~ N( mu , sigma / sqrt(n) )
* sample mean ~ t( mu , std / sqrt(n) )  degree of freedom is n - 1


When you think sample mean distributes in the area of 95% of this distribution,
lower_limit(using mu) < sample mean < upper_limit(using mu)

By transforming this inequality formula, you can get the formula like this,
lower_limit(using sample mean) < mu < upper_limit(using sample mean)

This is confidence interval.

(Confidence Interval)
* sample mean � Z * sigma / sqrt(n)
* sample mean � t_(df=n-1) * std / sqrt(n)


~~~ SAS
PROC TTEST data=asthma h0=3.25 alpha=0.05;
  VAR fev;
RUN;
~~~


Confidence Interval of pupolatoin proportion (one variable)
-----------------------------------------------------------

(Assumption)
* simple random sample
* np > 5, n(1-p) > 5

(Distribution)
1. #success ~ Bi ( n , p )
2. #success ~ N ( np, np(1-p) )
3. phat ~ N ( p , p(1-p) / n  )
  + Var(phat) = V( #success / n ) = np(1-p)/(n**2) = p(1-p)/n

Think about 95% area of this distribution.
lower(defined by p) < phat < upper (defined by p)

This can be transformed into
lower(defined by phat and p) < p < upper (defined by phat and p)

(Confidence Interval)
* p = phat � Z sqrt( p(1-p) / n )
* Wald CI : p -> phat 
  + p = phat � Z sqrt( phat (1-phat ) / n )
* Agresti-Coull
  + p = ptilda � Z sqrt( ptilda (1-ptilda) / n+4 )
  + ptilda = (x+2)/(n+4)


~~~ SAS
data coin;
input head count;
datalines;
 1 686
 0 1073
;
run;

proc freq data=coin;
weight count;
 tables head /binomial (level=1 p=0.4 all); 
 /* level = <num>  : The <n>th level of categorical value. Default is 1. The level is the order in table output.*/
 /* p= <prob>  : Probability of null hypothesis */
 /* all : requests all available types of confidence limits for the binomial proportion */
run;
~~~


Hypothesis Test (p value) Overview
----------------------------------

* Step1 : Hypothesis
* Step2 : Assumption
* Step3 : Calculate test statistics
* Step4 : Calculate p-value
* Step5 : Conclusion


Hypothesis Test for Population Mean
-----------------------------------

Suppose you test the hypothesis thta the population mean equals to 98.5. 
The samplesize is 130, the sample average is 98.24.. and the sample std is 0733.. 

Do the 5 Steps in SAS

~~~ SAS
/* Step1 Hypothesis */
/* H0 : mu = 98.5 */
/* HA : mu not= 98.5 */

/* Step2 Assumption */
/* Simple random sample. */
/* Cehck large enough sample. samplesize 130 > 30. */
PROC CONTENTS data=bodytemp;
RUN;
/* n : 130 */

/* Step3 Calculate test statistic. */
PROC MEANS data=bodytemp;
RUN;
/* mean : 98.2492308, std : 0.7331832 */

DATA step_3 ;
 t_statistic = (98.2492308 -  98.5 ) / ( 0.7331832 / sqrt(130));
PROC PRINT;
RUN;
/* t statistic => -3.89972*/

/* Step4 Calculate p-value. */
DATA step_4;
 set step_3;
 p_value = 2 * cdf("t", t_statistic , 130 - 1 ) ;
PROC PRINT;
RUN;
/* p value => 0.000154132  */
/* Step5 Conclusion.*/
/* At the 5% significant level, there is sufficient evidence to believe that the population mean body temperature is 98.5 degrees. */

~~~

Do the same thing in one code.

* PROC TTEST
  + option : h0=<>
  + option : alpha=<>
* Var statement in PROC  TEST

~~~
PROC TTEST data=bodytemp h0=98.5 alpha=0.05;
  VAR 
RUN;
~~~


Hypothesis Test for Population Proportion
-----------------------------------------

Suppose you test the hypothesis thta the population proportion equals to 0.01. 
The samplesize is 11034, the event proportion (risk) in this sample is 1.71%. 

Do the 5 Steps in SAS

~~~ SAS
/* Step1 Hypothesis */
/* H0 : p = 0.01 */
/* HA : p not= 0.01 */

/* Step2 Assumption */
/* Simple random sample */
/* Check n p > 5 , n (1 - p) > 5 */
PROC FREQ data=aspirin_ctrl;
  tables MI;
RUN;
/*Observation : 11034.  p = 0.01.  Therefore, n p > 5 , n (1 - p) > 5 (This p is parameter p.)*/

/* Step3 Calculate test statistic.*/
/* Total # of this sample : 11034 */
/* MI(+) in thi sample : 1.71 % */
DATA step3;
 /* (phat - p) / sqrt(p * (1-p)/n ) */
 z_value =( 0.0171 - 0.01 ) / sqrt( 0.01 * ( 1- 0.01) / 11034 );
PROC PRINT;
RUN;
/* z value is 7.49561. */

/* Step4 p value*/
DATA step4;
 set step3;
 p_value = 2 * (1 - cdf("normal", 7.49561 ) );
PROC PRINT;
RUN;
/* p value is 6.5947E-14 */

/* Step5 Conclusion. */
/* At the 5% siginificant level, there is sufficient evidence to believe the population percentaeg of MI among placebo group is different from 0.01.*/

~~~


Do the same thing in one code.

SAS code is the same as confidence interval for one variable proportion.
