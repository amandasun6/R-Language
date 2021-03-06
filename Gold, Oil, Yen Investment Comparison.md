# R-Language
#1  Check you working directory set the working directory

getwd()
setwd("~/Desktop/ANLY/RScript")


#2  Download goy data set. This dataset reperesnets daily prices of gold,
#   oil, and the price of 1 US dollar in terms of Japanese yen.
#   Set the first column in each data set to the date format 
#   and the remaining columns in numerical format.

library(readxl)
goy <- read_xls('goy.xls')  
goy$observation_date <- as.Date(goy$observation_date) 
str(goy)

#3  Create a new data set called "goycc" that contains all complete cases of goy data.
#   Utilize complete.cases function.

goycc <- goy[complete.cases(goy),]

#4 Create a stand alone variable "date" that takes on values of "observation_date"
# variable from the goycc data set. Set the mode of the variable to character

date <- as.character(goycc$observation_date)

#5 Find the range of dates covered in goycc data set by applying range function
#  to "date" variable. 

range(date)

#6 Create a time series object called "goyccts" by utilizing goycc dataset and 
#  ts() function. In this time series object please exclude the first column 
#  of the goycc dataset. 

df <- goycc[,-1]
goyccts <- ts(df, start = c(1971,1), end = c(2019,2), freq = 12)
str(goyccts)

#7 Reassign the value of the yen variable from the goyccts data set
#  by converting the exchange rate of yen that represents 
#  the price of 1 US Dollar in terms of Japanese yen to represent 
#  the price of 1 Yen in terms of US Dollar. 
#  This way if the number increases it represents appreciation of Yen. 
#  Hint: Reassign the value of yen variable by taking a reciprocal 

goyccts[,'yen'] <- 1/goyccts[,'yen']

#8 Plot the time series plot of the three assets. Do you see any trend?
# Do you see any seasonal component? Yes, the price of those three assets all got a rise in average price in the end of every years.

plot(goyccts)

#9 Utilize the aggregate function to plot annual average prices of the three assets.
#   How does this graph differ from the monthly time series plot? This one is more smooth and steady than the monthly time series plot.

plot(aggregate(goyccts))

#10 Find the average summer price of oil for the entire sample.

summeroilprice <- goycc[format.Date(goycc$observation_date, "%m") == '06' | format.Date(goycc$observation_date, "%m") == '07' | format.Date(goycc$observation_date, "%m") == '08',]
mean(summeroilprice$oil)

#11 Find the average winter price of oil for the entire sample.

winteroilprice <- goycc[format.Date(goycc$observation_date, "%m") == '12' | format.Date(goycc$observation_date, "%m") == '01' | format.Date(goycc$observation_date, "%m") == '02',]
mean(winteroilprice$oil)

#12 Find how the summer price of oil compares to the winter price of oil.
#   Please provide your answer in percentages. The price of summer is higher than winter around 7.24%

(mean(summeroilprice$oil)-mean(winteroilprice$oil))/mean(winteroilprice$oil)


#13 Use window() function to create three stand alone variables 
#   "gold", "oil", and "yen" that take on values of the "gold", "oil", and "yen" 
#   variables from the goyccts dataset starting from January of 2005

gold <- window(goyccts[,'gold'], start=c(2005,1))
oil <- window(goyccts[,'oil'], start=c(2005,1))
yen <- window(goyccts[,'yen'], start = c(2005,1))

#14 Use plot and decompose functions to generate three graphs that would depict
#   the observed values, trends, seasonal, and random components for "gold"
#   "oil" and "yen" variables. Would you choose multiplicative or 
#   additive decomposition model for each of the variables? I did not see much different between them two but I'd like to choose the decomposition model 
#because this one provide many details to help our analysis.

plot(gold)
plot(oil)
plot(yen)

gold_dec <- decompose(gold, type = 'additive')
plot(gold_dec)
oil_dec <- decompose(oil, type = 'additive')
plot(oil_dec)
yen_dec <- decompose(yen, type = 'additive')
plot(yen_dec)


#15 For each of the variables extract the random component and save 
#   them as "goldrand", "oilrand", and "yenrand". Moreover, use na.omit()
#   function to deal with the missing values.

goldrand <- na.omit(gold_dec$random)
oilrand <- na.omit(oil_dec$random)
yenrand <- na.omit(yen_dec$random)

#16 For the random component of each of the assets, please estimate 
#   autocorrelation function.Does any of the assets exhibit autocorrelation?
#   If yes, to what degree?  Yes, they all exhibit autocorrelation. They are showing the significant autocorrelation for lag value around over 0.5.
#   Keep in mind there are missing values. 

acf(goldrand)
acf(oilrand)
acf(yenrand)

#17 For all possible pairs of assets please estimate cross-correlation function 
#   Do any of the variable lead or precede each other?
#   Could you use any of the varibales to predict values of other variables?No,it could not use the variables to predict values of other variables.
#   Make sure to use detrended and seasonally adjusted variables. 
#   ("goldrand", "oilrand", and "yenrand")

ts.plot(goldrand, oilrand, col=c("orange","black"))
ts.plot(goldrand, yenrand, col=c("orange","black"))
ts.plot(oilrand, yenrand, col=c("orange","black"))
ccf(goldrand,oilrand)
ccf(goldrand, yenrand)
ccf(oilrand, yenrand)

#18 Based on the time series plot of gold, oil, and yen prices, 
#   there appears to be no systematic trends or seasonal effects. 
#   Therefore, it is reasonable to use exponential smoothing for these time series.
#   Estimate alpha, the smoothing parameter for gold, oil and yen. 
# They are all have a kind of smoothing with the alpha value around 0.999,while estimated mean of them are 1317.813562,60.4241433,9.188601e-03.

gold.holt <- HoltWinters(gold, beta = 0, gamma = 0); gold.holt
oil.holt <- HoltWinters(oil, beta = 0, gamma = 0); oil.holt
yen.holt <- HoltWinters(yen, beta = 0, gamma = 0); yen.holt


#19 Use plot() function to generate three graphs that depict observed 
#   and HoltWinter fitted values for each asset.

plot(gold.holt)
plot(oil.holt)
plot(yen.holt)

#20 Use window() function to create 3 new variables called 
#   "goldpre", "oilpre", and "yenpre" that covers the period from January 2005, 
#   until August 2018. 

goldpre <- window(gold, start = c(2005,1), end = c(2018,8)) 
oilpre <- window(oil, start = c(2005,1), end = c(2018,8)) 
yenpre <- window(yen, start = c(2005,1), end = c(2018,8)) 

#21 Use window() function to create 3 new variables called 
#   goldpost, oilpost, and yenpost that cover the period from September 2018, 
#   until February 2019.

goldpost <- window(gold, start = c(2018,9), end = c(2019,2)) 
oilpost <- window(oil, start = c(2018,9), end = c(2019,2)) 
yenpost <- window(yen, start = c(2018,9), end = c(2019,2)) 

#22 Estimate HoltWinters filter model for each asset, while using only only pre data.
#   Save each of these estimates as "gold.hw", "oil.hw", and "yen.hw".

gold.hw <- HoltWinters(goldpre, seasonal = 'additive')
oil.hw <- HoltWinters(oilpre, seasonal = 'additive')
yen.hw <- HoltWinters(yenpre, seasonal = 'additive')

#23 Use HoltWinters filter estimates generated in#23 and predict() function 
#   to create a 6 month ahead forecast of the gold, oil, and yen prices. 
#   Save these forcasted values as "goldforc", "oilforc", and "yenforc".

goldforc <-predict(gold.hw, n.ahead = 6)
oilforc <- predict(oil.hw, n.ahead = 6)
yenforc <- predict(yen.hw, n.ahead = 6)

#24 Use ts.plot() function to plot side-by-side post sample prices 
#   ("goldpost", "oilpost","yenpost") and their forecasted counterparts.
#   Please designate red color to represent the actual prices, 
#   and blue doted lines to represent forecasted values. 

par(mfrow = c(1,3))
plotgold <- ts.plot(goldpost, goldforc, lty = 1:2, col = c('red','blue'), ylim = c(1050,1320))
plotoil <- ts.plot(oilpost, oilforc,lty = 1:2, col = c('red','blue'))
plotyen <- ts.plot(yenpost, yenforc,lty = 1:2, col = c('red','blue'))

#25 calculate forecast mean percentage error for each assets forecasting model. 
#The oil with the lowest mean percentage error -7.141958

gold_per <- mean(((goldpost - goldforc)/goldpost)*100); gold_per
oil_per <- mean(((oilpost - oilforc)/oilpost)*100); oil_per
yen_per <- mean(((yenpost - yenforc)/yenpost)*100); yen_per

#26 Use gold, oil, and yen variables to estimate Holt-Winters model
#   for each asset. Save these estimates as "goldc.hw", "oilc.hw", and "yenc.hw".

goldc.hw <- HoltWinters(gold, seasonal = 'additive')
oilc.hw <- HoltWinters(oil, seasonal = 'additive')
yenc.hw <- HoltWinters(yen, seasonal = 'additive')


#27 Use "goldc.hw", "oilc.hw", and "yenc.hw" models to create an out-of-sample
#   forecasts to predict the prices of each of the assets for the rest of the 2019.
#   Save these forecasts as "goldforcos", "oilforcos", "yenforcos".

goldforcoc <-predict(goldc.hw, n.ahead = 10)
goldforcoc_10 <- goldforcoc['Nov']
oilforcoc <- predict(oilc.hw, n.ahead = 10)
yenforcoc <- predict(yenc.hw, n.ahead = 10)

# 28 Create time series plots for each asset, that combines the actual price data
#    of each asset and their out-of-sample forecasted values.
#Seems the prices of those three assets will going down by the end of the year.

par(mfrow = c(1,3))
plotfgold <- ts.plot(gold, goldforcoc, lty = 1:4, col = c('red','blue'))
plotfoil <- ts.plot(oil, oilforcoc,lty = 1:4, col = c('red','blue'))
plotfyen <- ts.plot(yen, yenforcoc,lty = 1:4, col = c('red','blue'))


# 29 Please calculate percentage change between the price of each asset in 
#    February 2019 and their forecasted December 2019 prices. 
#The Gold promises the highest rate of return

goldfeb <- window(gold, start = c(2019,2))[[1]]
golddec <- goldforcoc[[10]]
goldperc <- goldfeb/golddec; 
goldperc 
oilfeb <- window(oil, start = c(2019,2))[[1]]
oildec <- oilforcoc[[10]]
oilperc <- oilfeb/oildec; 
oilperc 
yenfeb <- window(yen, start = c(2019,2))[[1]]
yendec <- yenforcoc[[10]]
yenperc <- yenfeb/yendec; 
yenperc 
