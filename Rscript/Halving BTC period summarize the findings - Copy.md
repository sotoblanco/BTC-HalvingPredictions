---
title: "Halving BTC Period summarize the findings"
author: "Pastor Soto"
date: "10/21/2021"
output: 
  html_document: 
    code_folding: hide
---

```{r message=FALSE, warning=FALSE}
# load libraries
library(tidyverse)
library(plotly)
library(lubridate)
library(tidyquant)

options(scipen = 999) # use integer instead of scientific notation
```


## Daily results for halving period

```{r}
file_path = "C:/Users/Pastor/Dropbox/Pastor/data/barchart"
#data from firstratedata for crypto in UCT
btc <- read.csv(file.path(file_path, "BTCUSD.csv"))

btc <- plyr::rename(btc, c("Time"= "Date", "Last" = "Close"))

btc$Date <- as.Date(btc$Date, format = "%m/%d/%Y")

btc$day_month <- as.Date(btc$Date, format = "%Y-%m-%d")
btc <- btc[order(btc$Date),]

# get daily returns
btc$returns <- with(btc, log(Close/dplyr::lag(Close)))

# halving periods
btc <- btc %>% mutate(halving = case_when(Date < "2012-11-28" ~ "2010-07-20",
                                          Date >= "2012-11-28" & Date <= "2016-07-09" ~ "2012-11-28",
                                          Date > "2016-07-09" & Date <= "2020-05-11" ~ "2016-07-09",
                                          Date > "2020-05-11" ~ "2020-05-11"))


btc <- btc %>% group_by(halving) %>% mutate(num_halving = row_number())

```

```{r}

p <- btc %>% group_by(halving) %>% 
  ggplot(aes(day_month, Close, color = halving))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale")+
  xlab("Date")
ggplotly(p)


```

We explore the halving period from 2010 untill now. We identify an increasing trend with a ciclical behaviour.


```{r}
btc_2 <- btc %>%
  group_by(halving) %>%  # Need to group multiple stocks
  mutate(returns = if_else(num_halving == 1, 0, returns)) %>%
  mutate(cr = cumprod(1 + returns)) %>%
  mutate(cumulative_returns = cr - 1)

btc_2_scale <- btc_2 %>% group_by(num_halving) %>% mutate(scale_price = cr * 9446)


p <- btc_2_scale %>% group_by(halving) %>% 
  ggplot(aes(num_halving, scale_price, color = halving, label = day_month))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale standardize values")+
  xlab("Date")+
  theme_bw()
ggplotly(p)


```

The gaph shows the number of days between each halving period using the last halving as reference on a logaritmic scale. The results comparing the last havling period and the genesis block period suggest a match on the distribution with the first halving period, which would implie we already reach the exponential increase and we are in a downtrend, against this claim is the fact we already broke the first ATH which might suggest that we are waiting for the uptrend. 

By comparing the last halving with the first halving period we reach to a similar conclusion than the previous finding, however, since we broke the ATH there are reasons to belive we are waiting for the exponential peak on the price. 

The second halving is the one that has the most similar distribution to the last halving period, in which we already reach the peak and preparing for the last exponential move before the correction.


```{r}
p <- btc_2_scale %>% group_by(halving) %>% 
  ggplot(aes(day_month, cr, color = halving, label = Close))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale Standardize")+
  xlab("Date")+
  theme_bw()
ggplotly(p)


```

The halving periods clearly shows a major correction before the last exponatial movement with a decline in the expected return as the volatility on BTC decrease. From previous findigs the volatility has been experience a significant as the time goes by. 

## Monthly results for halving period BTC

```{r}

btc$month <- month(btc$Date)
btc$year <- year(btc$Date)

btc_month <- btc %>%  group_by(year, month) %>% summarise(Close = last(Close),
                                                          day_month = last(day_month),
                                                          halving = last(halving))
btc$returns <- with(btc, log(Close/dplyr::lag(Close)))

btc_month$returns <- with(btc_month, log(Close/dplyr::lag(Close)))

btc_month <- btc_month %>% group_by(halving) %>% mutate(num_halving = row_number())


btc_2_month <- btc_month %>%
  group_by(halving) %>%  # Need to group multiple stocks
  mutate(returns = if_else(num_halving == 1, 0, returns)) %>%
  mutate(cr = cumprod(1 + returns)) %>%
  mutate(cumulative_returns = cr - 1)


btc_2_month_scale <- btc_2_month %>% group_by(num_halving) %>% mutate(scale_price = cr * 9446)

```


```{r}

p <- btc_2_month_scale %>% group_by(halving) %>% 
  ggplot(aes(day_month, Close, color = halving))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale")+
  xlab("Date")
ggplotly(p)


```

Using BTC monthly data we identify the behavior of the trend in BTC in each halving period, which is similar to the first one in the aspect of a long wave before the final exponential movement before the downtrend. 


```{r}
p <- btc_2_month_scale %>% group_by(halving) %>% 
  ggplot(aes(num_halving, scale_price, color = halving, label = day_month))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale standardize values")+
  xlab("Date")+
  theme_bw()
ggplotly(p)


```

Using the halving periods on monthly prices of BTC we identfy the number of months on each halving, in which it is expected a similar trend with the second halving the next month followed by a major downtrend expected to happen faster than the previous halving.



```{r}
p <- btc_2_month_scale %>% group_by(halving) %>% 
  ggplot(aes(day_month, scale_price, color = halving, label = Close))+
  geom_line()+
  scale_y_continuous(trans = 'log10')+
  ggtitle("BTC Halving periods")+
  ylab("BTC Price (USD) Logarithmic scale")+
  xlab("Date")+
  theme_bw()
ggplotly(p)


```

Using date in the x-axis we see a similar pattern in all halving periods, the exponential move is lower on each halving and the downtrend after reaching the peak at the end of the year is stronger each halving, so we expect a bigger downtern after reaching the peak on the BTC price.

```{r}
btc_2_scale$year <- year(btc_2_scale$day_month)
btc_2_scale_p = btc_2_scale %>% group_by(year) %>% mutate(mean_Daily = mean(returns),
                                                sd_daily= sd(returns)) 

p = btc_2_scale_p %>% ggplot(aes(year, mean_Daily, color = halving, label = Close)) + geom_line()

ggplotly(p)

```



```{r}

p = btc_2_scale_p %>% ggplot(aes(year, sd_daily, color = halving, label = Close)) + geom_line()

ggplotly(p)

```

The results for the average return suggest a decreasing trend in the daily and monthly returns of BTC with a decrease on the volatility, by extrapolating these results we expect a decrease in the returns before the next halving and also a decrease in volatility which it is expected to occur closer to the halving. 


Github: https://github.com/sotoblanco/BTC-HalvingPredictions


Email: sotoblanco263542@gmail.com


Twitter: @PastorSotoB1
