# Visual_repos
my STATA repository
//my STATA workflow//
.................................
********************** Import Data ***********************************************
use "yourStataFile.dta", clear
load a dataset from the current directory
import delimited "yourFile.csv", /*rowrange(2:11) colrange(1:8) varnames(2)*/
import a .csv file
webuse set "https://github.com/GeoCenter/StataTraining/raw/master/Day2/Data"
webuse "wb_indicators_long"
set web-based directory and load data from the web
import excel "yourSpreadsheet.xlsx", /*sheet("Sheet1") cellrange(A2:H11) firstrow*/
import an Excel spreadsheet
Import Data
___________________________________________________________________________________
*****************************Explore Data ******************************************
*******************describe
describe make price   /*display variable type,format and any value/variable labels*/
*******************count 
countif price > 5000 /*number of rows (observations)Can be combined with logic*/
*******************summarize
summarize make price mpg /*print summary statistics(mean, stdev, min, max)for variables*/

******************Histogram
inspect mpg /*show histogram of data,number of missing or zero observations*/
histogram mpg, frequency /*plot a histogram of the distribution of a variable*/

******************codebook
codebook make price /*overview of variable type, stats,number of missing/unique values*/
display price[4] /*display the 4th observation in price; only works on single values*/
gsort price mpg (ascending) /*sort in order, first by price then miles per gallon*/
duplicates report /*finds all duplicate values in each variable*/
drop make /*remove the 'make' variable*/

*******************Transformation**********
reshape long coffee@ maize@, i(id) j(create new variable)
reshape wide coffee maize, i(country) j(year) 

drop if mpg < 20 drop in 1/4 /*drop observations based on a condition (left)or rows 1-4 (right)*/
keep if inrange(price, 5000, 10000) /*keep values of price between $5,000 – $10,000 (inclusive)*/
replace price = 5000 if price < 5000 /*replace all values of price that are less than $5,000 with 5000*/

*********sample 25
sample 25% of the observations in the dataset /*(use set seed # command for reproducible sampling)

*******************Tabulate*********/
tabulate agecategory
tabstat price weight mpg, by(foreign) stat(mean sd n) /*create compact table of summary statistics*/

************************generating variables*
=======================================================================================
******************************population characteristics***************
*SEX
gen sex=0 if gender=="Female"
replace sex=1 if gender =="Male"
label var sex "Female vrs Male"
label define gender 0 "Female" 1 "Male"
label value sex gender
tabulate sex

*AGEGROUP
gen agegroup=0 if age <5
replace agegroup=1 if age >=5 & age <15 
replace agegroup=2 if age >=15 & age <35 
replace agegroup=3 if age >=35 & age <=45 
replace agegroup=4 if age >45
label var agegroup "under five vrs children vrs youth vrs young adult vrs old adult" 
label define agecategory 0 "under five" 1 "children" 2 "youth" 3 "young adult" 4 "old adult"
label value agegroup agecategory

*AGECATEGORY
gen agecategory=0 if age <5
replace agecategory=1 if age >=5 & age <15
replace agecategory=2 if age >=15 & age <35
replace agecategory=3 if age >=35 & age <=45
replace agecategory=4 if age >45
label var agecategory "0-4 vs 5-14 vs 15-34 vs 35-44 vs 45-100"
label define agerange 0 "0-4" 1 "5-14" 2 "15-34" 3 "35-44" 4 "45-100"
label value agecategory agerange
tabulate agecategory

*EDUCATION
gen education=0 if motherseducationlevel=="None"
replace education=1 if motherseducationlevel=="primary" 
replace education=2 if motherseducationlevel=="Junior High" 
replace education=3 if motherseducationlevel =="Senior High" 
replace education=4 if motherseducationlevel=="Tertiary"
label var education "None vrs primary vrs Junior Hgh vrs Senior Hgh vrs Tertiary"
label define literacy 0 "None" 1 "primary" 2 "Junior High" 3 "Senior High" 4 "Tertiary"
label value education literacy
tabulate education

*INSURANCE
gen insurance=0 if nhis=="No" 
replace insurance=1 if nhis=="Yes"
label var insurance "NO vrs Yes"
label define coverage 0 "No" 1 "Yes"
label value insurance coverage
tabulate insurance
*****************continious data--------------
sum distance
sum travel_time

--------------------------------------------------------------------------------
**************************** Data Analysis *************************************

univar price mpg, boxplot /*calculate univariate summary with box-and-whiskers plot*/
stem mpg /*return stem-and-leaf display of mpg*/
summarize price mpg, detail /*calculate a variety of univariate summary statistics*/
ci mean mpg price, level(99) /*compute standard errors and confidence intervals*/
correlate mpg price /*return correlation or covariance matrix*/
pwcorr price mpg weight, star(0.05) /*return all pairwise correlation coefficients with sig. levels*/
mean price mpg /*estimates of means, including standard errors*/
proportion rep78 foreign /*estimates of proportions, including standard errors for categories identified in varlist*/
ratio price/mpg /*estimates of ratio, including standard errors*/
total price /*estimates of totals, including standard errors*/

********STATISTICS==============
tabulate foreign rep78, chi2 exact expected /*tabulate foreign and repair record and 
return chi2and Fisher’s exact statistic alongside the expected values*/
ttest mpg, by(foreign) /*estimate t test on equality of means for mpg by foreign*/
prtest foreign == 0.5 /*one-sample test of proportions*/
ksmirnov mpg, by(foreign) exact /*Kolmogorov–Smirnov equality-of-distributions test*/
ranksum mpg, by(foreign) /*equality tests on unmatched data (independent samples)*/
anova systolic drug webuse systolic, clear /*analysis of variance and covariance*/
pwmean mpg, over(rep78) pveffects mcompare(tukey) /*estimate pairwise comparisons of means 
with equalvariances include multiple comparison adjustment*/

********FIT models==============================================================
regress price mpg weight, vce(robust) /*fit ordinary least-squares (OLS) model on mpg, weight, 
and foreign, apply robust standard errors*/
regress price mpg weight if foreign == 0, vce(cluster rep78) /*regress price only on domestic cars, 
cluster standard errors*/
rreg price mpg weight, genwt(reg_wt) /*estimate robust regression to eliminate outliers*/
probit foreign turn price, vce(robust) /*estimate probit regression with robust standard errors*/
logit foreign headroom mpg, or /*estimate logistic regression and report odds ratios*/

******** TIME SERIES============================================================
tsset time, yearly /*declare sunspot data to be yearly time series*/
tsreport /*report time-series aspects of a dataset*/
generate lag_spot = L1.spot /*create a new variable of annual lags of sunspots*/
tsline spot /*plot time series of sunspots*/
arima spot, ar(1/2) /*fit an autoregressive model with 2 lags*/

******* Survival Analysis=======================================================
stset studytime, failure(died) /*declare survey design for a dataset*/
stsum /*summarize survival-time data*/
stcox drug age /*fit a Cox proportional hazards model*/

******** PANEL / LONGITUDINAL===================================================
xtset id year /*declare national longitudinal data to be a panel*/
xtdescribe /*report panel aspect of a dataset*/
xtsum hours /*summarize hours worked, decomposing standard deviation into between and within components*/
xtline ln_wage if id <= 22, tlabel(#3)/*plot panel data as a line plot*/
xtreg ln_w c.age##c.age ttl_exp, fe vce(robust)/*fit a fixed-effects model with robust standard errors*/

**********SURVEY DATA===========================================================
svyset psuid [pweight = finalwgt], strata(stratid) /*declare survey design for a dataset*/
svydescribe /*report survey-data details*/
svy: mean age, over(sex) /*estimate a population mean for each subpopulation*/
svy, subpop(rural): mean age /*estimate a population mean for rural areas*/
svy: tabulate sex heartatk /*report two-way table with tests of independence*/
svy: reg zinc c.age##c.age female weight rural /*estimate a regression using survey weights*/

**********Diagnostics===========================================================
estat hettest /*test for heteroskedasticity*/
ovtest /*test for omitted-variable bias*/
vif /*report variance inflation factor*/

**********Data Visualization====================================================
*----------------ONE VARIABLE----------------
histogram mpg, width(5) freq kdensity kdenopts(bwidth(5))/*CONTINUOUS*/
kdensity mpg, bwidth(3) 
graph bar (count), over(foreign, gap(*0.5)) intensity(*0.5) /*DISCRETE*/
graph bar (percent), over(rep78) over(foreign)  /*grouped barplot*/

*----------------DISCRETE X, CONTINUOUS Y------
graph bar (median) price, over(foreign)
graph dot (mean) length headroom, over(foreign) m(1, ms(S))
graph hbox mpg, over(rep78, descending) by(foreign) missing
vioplot price, over(foreign)

*--------------- TWO CONTINUOUS variable-------  
graph matrix mpg price weight, half
twoway scatter mpg weight, jitter(7)
twoway scatter mpg weight, mlabel(mpg)
twoway connected mpg price, sort(price)
twoway area mpg price, sort(price)
twoway bar price rep78
twoway dot mpg rep78
twoway dropline mpg price in 1/5
twoway rcapsym length headroom price
twoway rarea length headroom price, sort
twoway rbar length headroom price
twoway pcspike wage68 ttl_exp68 wage88 ttl_exp88
twoway pccapsym wage68 ttl_exp68 wage88 ttl_exp88

*----------------SUMMARY PLOTS----------------
twoway mband mpg weight || scatter mpg weight
binscatter weight mpg, line(none)
twoway lfitci mpg weight || scatter mpg weight
twoway lowess mpg weight || scatter mpg weight
twoway qfitci mpg weight, alwidth(none) || scatter mpg weight

*--------------Saving plot-------------------
graph save "myPlot.gph"
---------------------------------------------
sum  residence
poisson death male female
poisson death belowfive children youth youngadult oldadult
poisson death none primary juniorhigh seniorhigh tertiary

poisson death insured noninsured
poisson death farming trader artisan teaching mining businessman foodvendor driver student
poisson death transport_availability 

poisson death distance
poisson death traveltime
poisson death residence
