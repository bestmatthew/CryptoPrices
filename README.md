[top photo will go heeeeeerrrreeeee]

### CryptoPrices
Exploratory Data Analysis around BTC and other cryptocurrencies

## Introduction

Cryptocurrency, a digital asset designed to work as a medium of exchange, is far from a new concept. In 1983 eCash was a published & established concept: eCash software on a user's machine could store digital money which was cryptographically signed by a bank. The user could then spend the digital money at an accepting vendor without having to open any vendor accounts or sharing any payment information like credit card numbers. By 1990 eCash was 'off-line' meaning payees no longer had to contact a third party (i.e, The Bank). Exactly one (1) U.S. Bank accepted eCash as a micropayment system, before scrapping it completely a few years later, when it was picked up by Credit Suisse, Deutsche Bank, Bank Austria, and others until eCash went bankrupt in 1998, unable to compete with credit cards.

When Bitcoin was released over a decade later in 2009, it was the world's first decentralized crypocurrency -- lacking an organizational or authoritative controlling body -- and put simply, an asset which may only be accessed by its key-holder.

The history lesson ends here, however.

We will be examining price data from Bitcoin/BTC (2009) along with a few others that emerged years afterwards: Litecoin/LTC (2011, Ripple/XRP (2013), Monero/XMR (2014), and Etherium/ETH (2015). To state the obvious, this is in no way a *comparison* across these cryptocurrencies. Their only similarities, for the purposes of this exercise, is that they're digital assets that have a fluctuating USD value.

We will examine the individual price data and their movement to look for correlations beween them.

## The Data

Because most crypocurrencies are bound to a blockchain - a public ledger of records - there is a considerable amount of data available for examination. The dearth of data held by the [Blockchain.info API](https://www.blockchain.com/charts) was too much for now but may hopefully provide more data for future projects.
We used a cleaned dataset from [Sudalai Raj Kumar](https://www.kaggle.com/sudalairajkumar/cryptocurrencypricehistory) which provides a clean daily breakdown of Open/Close prices, Highs/Lows, daily trade volume and market cap among 20+ cryptocurencies. These simple, straightforward, value-focused metrics should hopefully be easy to digest and allow us to really understand our dataset.

## Data Exploration

- First we'll make the necessary imports and read in the data.

```python
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
```

```python
btc = pd.read_csv('coin_Bitcoin.csv')  #read each csv data file into a pandas df
btc['Swing'] = btc['High'] - btc['Low']  #adding a col "Swing" to show the difference between the daily high/low
btc['Percent'] = btc['High'] / btc['Low'] #adding another col "Percent" to show the quotient of daily high/low prices
btc                                        # variable for the Bitcoin pandas df with its added columns is called
```

![snapbtcdf](img/x1.png)

- A look at the Bitcoin dataframe is shown above. You'll see two indexes (0- and 1-index), 'Name', 'Symbol', 'Date', 'High'/'Low', 'Open'/'Close', 'Volume', & 'Marketcap', along with the daily 'Swing' and its corresponding 'Percent' now ready for examination. This pandas dataframe contains 2862 rows ?? 12 columns holding daily data from 04/29/2013 to 2/27/2021.

```python
coins = btc, ltc, xrp, xmr, eth      #variable that holds all data for the 5 dataframes

coins
```

![pic2](img/x2.png)


- With dataframes for BTC, LTC, XRP, XMR, & ETH loaded, our analysis can begin. First thing we examined was the average price points. We looked at the average highs and the lows, examining their variance and giving us an initial look into our different cryptocurrencies.

```python
def getavgl(data):
    avgs = []
    for x in data:
        avgs.append(np.mean(x['Low']))
    return avgs

def getavgh(data):
    avgs = []
    for x in data:
        avgs.append(np.mean(x['High']))
    return avgs
```

```python
avglows = np.array(getavgl(coins))
avghighs = np.array(getavgh(coins))
swings = avghighs/avglows
avglows, \
avghighs, \
swings
```

![swingsarr](img/x3.png)

- Ignoring the clear, large price differences, we can already see clear variances in their respective volatilities, with Ripple/XRP having the most variation between average highs/lows (10%) with Bitcoin/BTC having the least variation from average highs/lows (5%). Interesting... isn't Bitcoin supposed to be the "wildly unstable" asset of the bunch?

![barshighlowvar](img/varbarchar.png)

- Let's take another angle. Let's examine each days' 'Open' and 'Close' value and note if 'Close' is higher or lower than its 'Open'. We like making money, so lets find out who's been celebrating each night, and who's been crying?

```python

def higherlower(df):
    higher = 0  #counting days where the price was higher at close than at open
    lower = 0   #counting days where the price was lower at close than at open
    for index, row in df.iterrows():
        if df.loc[index,'Close'] > df.loc[index,'Open']:
            higher += 1
        elif df.loc[index,'Close'] == df.loc[index,'Open']: #ignore this unlikely scenario
            pass
        else:
            lower +=1
    return higher, lower, higher/lower

```
```python
a = higherlower(btc)
b = higherlower(ltc)
c = higherlower(xmr)
d = higherlower(xrp)
e = higherlower(eth)

a,b,c,d,e
```
![outputarrayhighlow](img/x5.png)

![higherlowertotalbar](img/x4.png)


- Several things become fairly apparent here. From this data we can see Bitcoin/BTC has the highest rate of "higher" days while Ripple/XRP has the least of the group. Note that this dataset shows 1 winner, 4 losers.
- The next biggest and most obvious is regarding the data; the price have start dates ranging from 2013 to 2015, if we're going to start examining the prices and their movements, we should probably trim everything down so that each row of each dataframe corresponds to the same date.
    - In order to do this, we have to trim down to size of our shortest list, our newest crypocurrency, Etherium, where the data begins collection on 08/08/2015 and contains 2031 rows. This is where we will continue working.

```python
# lets trim all our df down to magic len=2031.
btctrim= btc['Low'].iloc[831:]
ltctrim = ltc['Low'].iloc[831:]
xrptrim = xrp['Low'].iloc[733:]
xmrtrim = xmr['Low'].iloc[442:]
ethtrim = eth['Low']
```

- Are these cryptocurrencies related? How closely do they move together? We examined the "daily delta" of the daily high price / daily low to see if volatilities may be related. The Pearson r values are shown below.



BTC              | LTC         | XRP         | XMR          | ETH
---------------- | ----------- | ----------- | ------------ | ---
BTC|	1.000000 |	0.699078  |	0.396935   |	0.504828  |	0.388574
LTC|	0.699078 |	1.000000  |	0.514094   |	0.419860  |	0.334974
XRP|	0.396935 |	0.514094  |	1.000000   |	0.298468  |	0.230860
XMR|	0.504828 |	0.419860  |	0.298468   |	1.000000  |	0.389764
ETH|	0.388574 |	0.334974  |	0.230860   |	0.389764  |	1.000000


![CorrelationHeatmap](img/dailydeltamatrix.png)

- The correlations, overall, are alarmingly high. The lowest correlations, ETH/XRP (r = 0.230860) still fairly strong, with three (3) currencies with r > .5, *very high*: BTC/LTC(r = 0.699078), BTC/XMR (r = 0.504828), LTC/XRP (r = 0.514094).

- Let's "zoom out" in a way, and actually see the data in our correlation matrix. Below is a visualization of 2031 datapoints spanning 7 years across each of the 5 cryptocurrencies, starting in August 2015 - Feb 2021. Each datapoint represents the "daily delta" by examining the daily high/low as a percentage. The higher the plot, the wilder the day. Instantly you may identify the most volitile days and most volatile currencies by examining the outlying marks.

![Scatter5coins](img/scatter5coins.png)

- It's a little busy to look at 10,000 dots at once, but seeing the volatility over the 2000 days brings more questions than it answers, but some more history may be needed for context. 2017 had outrageous growth across the board (BTC in Jan 2017: $800 ; BTC in Dec 2017: $18,000) and its reflected in our plot. Interesting clusters of volatility can be earily be observed in late 2016, 2017, late 2019, and the giant losses, thank you COVID, can be seen about 30% (March-April) into 2020...

- Lets take out a bit of this noise. We can see the most volatile of the bunch are XRP, XMR, and ETH. A plot of just those:

![top3deltas](img/top3deltachart.png)