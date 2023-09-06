# Quantitative Momentum Strategy
## -Chayan Roy

"Momentum investing" means investing in the stocks that have increased in price the most.

For this project, we're going to build an investing strategy that selects the 50 stocks with the highest price momentum. From there, we will calculate recommended trades for an equal-weight portfolio of these 50 stocks.




## Library Imports

First we will import the open-source software libraries that we'll be using in this project.


```python
import numpy as np #The Numpy numerical computing library
import pandas as pd #The Pandas data science library
import requests #The requests library for HTTP requests in Python
import xlsxwriter #The XlsxWriter libarary for 
import math #The Python math module
from scipy import stats #The SciPy stats module
```

## Importing Our List of Stocks

Here we'll import our list of stocks and API token for IEX cloud platform. API token is saved as variable in secrets.py file. During version controll this file is not uploaded making your token secrect and secure.



```python
stocks = pd.read_csv('sp_500_stocks.csv')
from secrets import IEX_CLOUD_API_TOKEN
```

## This is a sample API Call

Here we are using sandbox version of iex cloud. This is a sample api call where details of one stock, APPLE is loaded.


```python
symbol = 'AAPL'
api_url = f'https://sandbox.iexapis.com/stable/stock/{symbol}/stats?token={IEX_CLOUD_API_TOKEN}'
data = requests.get(api_url).json()
data
```




    {'week52change': 1.271858,
     'week52high': 462.02,
     'week52low': 206.85,
     'marketcap': 1937104274094,
     'employees': 137265,
     'day200MovingAvg': 309.44,
     'day50MovingAvg': 390.09,
     'float': 4440629054,
     'avg10Volume': 54435185.2,
     'avg30Volume': 39067154.1,
     'ttmEPS': 13.7084,
     'ttmDividendRate': 3.22,
     'companyName': 'Apple, Inc.',
     'sharesOutstanding': 4331609946,
     'maxChangePercent': 452.5766,
     'year5ChangePercent': 3.0546,
     'year2ChangePercent': 1.1867,
     'year1ChangePercent': 1.186376,
     'ytdChangePercent': 0.512578,
     'month6ChangePercent': 0.407457,
     'month3ChangePercent': 0.485051,
     'month1ChangePercent': 0.19254,
     'day30ChangePercent': 0.253108,
     'day5ChangePercent': -0.008155,
     'nextDividendDate': '2020-08-16',
     'dividendYield': 0.007235663525925464,
     'nextEarningsDate': '2020-10-17',
     'exDividendDate': '2020-08-06',
     'peRatio': 34.17,
     'beta': 1.15885673879414}




```python
data['year1ChangePercent']
```




    1.186376



## Executing A Batch API Call & Building Our DataFrame in Pandas

Now we execute several batch API calls and add the information we need to our DataFrame.

It contains a function called `chunks` that we can use to divide our list of securities into groups of 100 because using batch api call we can get details of atmost 100 stocks at a time.


```python
# Function sourced from 
# https://stackoverflow.com/questions/312443/how-do-you-split-a-list-into-evenly-sized-chunks
def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]   
        
symbol_groups = list(chunks(stocks['Ticker'], 100))
symbol_strings = []
for i in range(0, len(symbol_groups)):
    symbol_strings.append(','.join(symbol_groups[i]))
#     print(symbol_strings[i])

my_columns = ['Ticker', 'Price', 'One-Year Price Return', 'Number of Shares to Buy']
```

Now we need to create a blank DataFrame and add our data to the data frame one-by-one.


```python
final_dataframe = pd.DataFrame(columns = my_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=stats,quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        final_dataframe = final_dataframe.append(
                                        pd.Series([symbol, 
                                                   data[symbol]['quote']['latestPrice'],
                                                   data[symbol]['stats']['year1ChangePercent'],
                                                   'N/A'
                                                   ], 
                                                  index = my_columns), 
                                        ignore_index = True)
        
    
final_dataframe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>101.50</td>
      <td>0.452986</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>13.65</td>
      <td>-0.527621</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>157.72</td>
      <td>0.088479</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>453.87</td>
      <td>1.172528</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>95.71</td>
      <td>0.476493</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>93.50</td>
      <td>-0.208066</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>138.91</td>
      <td>0.003031</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>287.51</td>
      <td>0.369427</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>35.73</td>
      <td>-0.162236</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>165.46</td>
      <td>0.288770</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 4 columns</p>
</div>



## Removing Low-Momentum Stocks

The investment strategy that we're building seeks to identify the 50 highest-momentum stocks in the S&P 500.

Because of this, the next thing we need to do is remove all the stocks in our DataFrame that fall below this momentum threshold. We'll sort the DataFrame by the stocks' one-year price return, and drop all stocks outside the top 50.



```python
final_dataframe.sort_values('One-Year Price Return', ascending = False, inplace = True)
final_dataframe = final_dataframe[:51]
final_dataframe.reset_index(drop = True, inplace = True)
final_dataframe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NVDA</td>
      <td>458.14</td>
      <td>2.015964</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>DXCM</td>
      <td>441.29</td>
      <td>1.721219</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AMD</td>
      <td>85.46</td>
      <td>1.632817</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CARR</td>
      <td>29.94</td>
      <td>1.489900</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AAPL</td>
      <td>453.87</td>
      <td>1.172528</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>5</th>
      <td>REGN</td>
      <td>624.50</td>
      <td>1.031728</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>6</th>
      <td>SWKS</td>
      <td>153.26</td>
      <td>0.921570</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WST</td>
      <td>283.20</td>
      <td>0.921000</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>8</th>
      <td>LRCX</td>
      <td>393.68</td>
      <td>0.892740</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>9</th>
      <td>QRVO</td>
      <td>132.38</td>
      <td>0.858187</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>10</th>
      <td>PYPL</td>
      <td>196.30</td>
      <td>0.829338</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>11</th>
      <td>AMZN</td>
      <td>3192.88</td>
      <td>0.759898</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ATVI</td>
      <td>83.82</td>
      <td>0.710587</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ALGN</td>
      <td>313.73</td>
      <td>0.709093</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NEM</td>
      <td>63.72</td>
      <td>0.698936</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ODFL</td>
      <td>196.24</td>
      <td>0.690263</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>16</th>
      <td>ROL</td>
      <td>55.90</td>
      <td>0.676072</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>17</th>
      <td>NOW</td>
      <td>439.05</td>
      <td>0.656782</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>18</th>
      <td>LOW</td>
      <td>159.19</td>
      <td>0.646186</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>19</th>
      <td>DPZ</td>
      <td>405.67</td>
      <td>0.644884</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>20</th>
      <td>FBHS</td>
      <td>85.93</td>
      <td>0.635603</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>21</th>
      <td>FAST</td>
      <td>49.98</td>
      <td>0.624303</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>22</th>
      <td>TGT</td>
      <td>136.10</td>
      <td>0.616877</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>23</th>
      <td>QCOM</td>
      <td>119.34</td>
      <td>0.609546</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>24</th>
      <td>URI</td>
      <td>181.40</td>
      <td>0.590406</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>25</th>
      <td>CHTR</td>
      <td>602.51</td>
      <td>0.586074</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>26</th>
      <td>MSCI</td>
      <td>369.93</td>
      <td>0.583839</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>27</th>
      <td>VAR</td>
      <td>174.79</td>
      <td>0.569642</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>28</th>
      <td>ROK</td>
      <td>247.30</td>
      <td>0.560821</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>29</th>
      <td>BIO</td>
      <td>517.62</td>
      <td>0.558503</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>30</th>
      <td>KSU</td>
      <td>193.03</td>
      <td>0.548382</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>31</th>
      <td>KLAC</td>
      <td>216.00</td>
      <td>0.547432</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>32</th>
      <td>ADBE</td>
      <td>449.01</td>
      <td>0.540159</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>33</th>
      <td>ABMD</td>
      <td>307.07</td>
      <td>0.537076</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>34</th>
      <td>CDNS</td>
      <td>111.60</td>
      <td>0.532542</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>35</th>
      <td>NFLX</td>
      <td>486.12</td>
      <td>0.531236</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>36</th>
      <td>ADSK</td>
      <td>240.90</td>
      <td>0.526803</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>37</th>
      <td>FTNT</td>
      <td>132.50</td>
      <td>0.522105</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>38</th>
      <td>MSFT</td>
      <td>215.73</td>
      <td>0.518900</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>39</th>
      <td>EA</td>
      <td>145.30</td>
      <td>0.517087</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>40</th>
      <td>TMO</td>
      <td>422.95</td>
      <td>0.513970</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>41</th>
      <td>KR</td>
      <td>36.29</td>
      <td>0.502626</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>42</th>
      <td>DHI</td>
      <td>73.64</td>
      <td>0.499570</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>43</th>
      <td>CTXS</td>
      <td>141.53</td>
      <td>0.496854</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>44</th>
      <td>LEN</td>
      <td>77.72</td>
      <td>0.495581</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>45</th>
      <td>MAS</td>
      <td>61.01</td>
      <td>0.493611</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>46</th>
      <td>TMUS</td>
      <td>119.48</td>
      <td>0.490167</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>47</th>
      <td>BBY</td>
      <td>107.33</td>
      <td>0.486775</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>48</th>
      <td>VRTX</td>
      <td>280.26</td>
      <td>0.486553</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>49</th>
      <td>PWR</td>
      <td>50.22</td>
      <td>0.482556</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>50</th>
      <td>OTIS</td>
      <td>67.90</td>
      <td>0.481000</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
</div>



## Calculating the Number of Shares to Buy

We now need to calculate the number of shares we need to buy. The one change we're going to make is wrapping this functionality inside a function, since we'll be using it again later in this Jupyter Notebook.



```python
def portfolio_input():
    global portfolio_size
    portfolio_size = input("Enter the value of your portfolio:")

    try:
        val = float(portfolio_size)
    except ValueError:
        print("That's not a number! \n Try again:")
        portfolio_size = input("Enter the value of your portfolio:")

portfolio_input()
print(portfolio_size)
```

    Enter the value of your portfolio:1000000
    1000000



```python
position_size = float(portfolio_size) / len(final_dataframe.index)
for i in range(0, len(final_dataframe['Ticker'])):
    final_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / final_dataframe['Price'][i])
final_dataframe
```

    /Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/pandas/core/indexing.py:494: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self.obj[item] = s





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Ticker</th>
      <th>Price</th>
      <th>One-Year Price Return</th>
      <th>Number of Shares to Buy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NVDA</td>
      <td>458.14</td>
      <td>2.015964</td>
      <td>42</td>
    </tr>
    <tr>
      <th>1</th>
      <td>DXCM</td>
      <td>441.29</td>
      <td>1.721219</td>
      <td>44</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AMD</td>
      <td>85.46</td>
      <td>1.632817</td>
      <td>229</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CARR</td>
      <td>29.94</td>
      <td>1.489900</td>
      <td>654</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AAPL</td>
      <td>453.87</td>
      <td>1.172528</td>
      <td>43</td>
    </tr>
    <tr>
      <th>5</th>
      <td>REGN</td>
      <td>624.50</td>
      <td>1.031728</td>
      <td>31</td>
    </tr>
    <tr>
      <th>6</th>
      <td>SWKS</td>
      <td>153.26</td>
      <td>0.921570</td>
      <td>127</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WST</td>
      <td>283.20</td>
      <td>0.921000</td>
      <td>69</td>
    </tr>
    <tr>
      <th>8</th>
      <td>LRCX</td>
      <td>393.68</td>
      <td>0.892740</td>
      <td>49</td>
    </tr>
    <tr>
      <th>9</th>
      <td>QRVO</td>
      <td>132.38</td>
      <td>0.858187</td>
      <td>148</td>
    </tr>
    <tr>
      <th>10</th>
      <td>PYPL</td>
      <td>196.30</td>
      <td>0.829338</td>
      <td>99</td>
    </tr>
    <tr>
      <th>11</th>
      <td>AMZN</td>
      <td>3192.88</td>
      <td>0.759898</td>
      <td>6</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ATVI</td>
      <td>83.82</td>
      <td>0.710587</td>
      <td>233</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ALGN</td>
      <td>313.73</td>
      <td>0.709093</td>
      <td>62</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NEM</td>
      <td>63.72</td>
      <td>0.698936</td>
      <td>307</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ODFL</td>
      <td>196.24</td>
      <td>0.690263</td>
      <td>99</td>
    </tr>
    <tr>
      <th>16</th>
      <td>ROL</td>
      <td>55.90</td>
      <td>0.676072</td>
      <td>350</td>
    </tr>
    <tr>
      <th>17</th>
      <td>NOW</td>
      <td>439.05</td>
      <td>0.656782</td>
      <td>44</td>
    </tr>
    <tr>
      <th>18</th>
      <td>LOW</td>
      <td>159.19</td>
      <td>0.646186</td>
      <td>123</td>
    </tr>
    <tr>
      <th>19</th>
      <td>DPZ</td>
      <td>405.67</td>
      <td>0.644884</td>
      <td>48</td>
    </tr>
    <tr>
      <th>20</th>
      <td>FBHS</td>
      <td>85.93</td>
      <td>0.635603</td>
      <td>228</td>
    </tr>
    <tr>
      <th>21</th>
      <td>FAST</td>
      <td>49.98</td>
      <td>0.624303</td>
      <td>392</td>
    </tr>
    <tr>
      <th>22</th>
      <td>TGT</td>
      <td>136.10</td>
      <td>0.616877</td>
      <td>144</td>
    </tr>
    <tr>
      <th>23</th>
      <td>QCOM</td>
      <td>119.34</td>
      <td>0.609546</td>
      <td>164</td>
    </tr>
    <tr>
      <th>24</th>
      <td>URI</td>
      <td>181.40</td>
      <td>0.590406</td>
      <td>108</td>
    </tr>
    <tr>
      <th>25</th>
      <td>CHTR</td>
      <td>602.51</td>
      <td>0.586074</td>
      <td>32</td>
    </tr>
    <tr>
      <th>26</th>
      <td>MSCI</td>
      <td>369.93</td>
      <td>0.583839</td>
      <td>53</td>
    </tr>
    <tr>
      <th>27</th>
      <td>VAR</td>
      <td>174.79</td>
      <td>0.569642</td>
      <td>112</td>
    </tr>
    <tr>
      <th>28</th>
      <td>ROK</td>
      <td>247.30</td>
      <td>0.560821</td>
      <td>79</td>
    </tr>
    <tr>
      <th>29</th>
      <td>BIO</td>
      <td>517.62</td>
      <td>0.558503</td>
      <td>37</td>
    </tr>
    <tr>
      <th>30</th>
      <td>KSU</td>
      <td>193.03</td>
      <td>0.548382</td>
      <td>101</td>
    </tr>
    <tr>
      <th>31</th>
      <td>KLAC</td>
      <td>216.00</td>
      <td>0.547432</td>
      <td>90</td>
    </tr>
    <tr>
      <th>32</th>
      <td>ADBE</td>
      <td>449.01</td>
      <td>0.540159</td>
      <td>43</td>
    </tr>
    <tr>
      <th>33</th>
      <td>ABMD</td>
      <td>307.07</td>
      <td>0.537076</td>
      <td>63</td>
    </tr>
    <tr>
      <th>34</th>
      <td>CDNS</td>
      <td>111.60</td>
      <td>0.532542</td>
      <td>175</td>
    </tr>
    <tr>
      <th>35</th>
      <td>NFLX</td>
      <td>486.12</td>
      <td>0.531236</td>
      <td>40</td>
    </tr>
    <tr>
      <th>36</th>
      <td>ADSK</td>
      <td>240.90</td>
      <td>0.526803</td>
      <td>81</td>
    </tr>
    <tr>
      <th>37</th>
      <td>FTNT</td>
      <td>132.50</td>
      <td>0.522105</td>
      <td>147</td>
    </tr>
    <tr>
      <th>38</th>
      <td>MSFT</td>
      <td>215.73</td>
      <td>0.518900</td>
      <td>90</td>
    </tr>
    <tr>
      <th>39</th>
      <td>EA</td>
      <td>145.30</td>
      <td>0.517087</td>
      <td>134</td>
    </tr>
    <tr>
      <th>40</th>
      <td>TMO</td>
      <td>422.95</td>
      <td>0.513970</td>
      <td>46</td>
    </tr>
    <tr>
      <th>41</th>
      <td>KR</td>
      <td>36.29</td>
      <td>0.502626</td>
      <td>540</td>
    </tr>
    <tr>
      <th>42</th>
      <td>DHI</td>
      <td>73.64</td>
      <td>0.499570</td>
      <td>266</td>
    </tr>
    <tr>
      <th>43</th>
      <td>CTXS</td>
      <td>141.53</td>
      <td>0.496854</td>
      <td>138</td>
    </tr>
    <tr>
      <th>44</th>
      <td>LEN</td>
      <td>77.72</td>
      <td>0.495581</td>
      <td>252</td>
    </tr>
    <tr>
      <th>45</th>
      <td>MAS</td>
      <td>61.01</td>
      <td>0.493611</td>
      <td>321</td>
    </tr>
    <tr>
      <th>46</th>
      <td>TMUS</td>
      <td>119.48</td>
      <td>0.490167</td>
      <td>164</td>
    </tr>
    <tr>
      <th>47</th>
      <td>BBY</td>
      <td>107.33</td>
      <td>0.486775</td>
      <td>182</td>
    </tr>
    <tr>
      <th>48</th>
      <td>VRTX</td>
      <td>280.26</td>
      <td>0.486553</td>
      <td>69</td>
    </tr>
    <tr>
      <th>49</th>
      <td>PWR</td>
      <td>50.22</td>
      <td>0.482556</td>
      <td>390</td>
    </tr>
    <tr>
      <th>50</th>
      <td>OTIS</td>
      <td>67.90</td>
      <td>0.481000</td>
      <td>288</td>
    </tr>
  </tbody>
</table>
</div>



## Building a Better (and More Realistic) Momentum Strategy

Real-world quantitative investment firms differentiate between "high quality" and "low quality" momentum stocks:

* High-quality momentum stocks show "slow and steady" outperformance over long periods of time
* Low-quality momentum stocks might not show any momentum for a long time, and then surge upwards.


To identify high-quality momentum, we're going to build a strategy that selects stocks from the highest percentiles of: 

* 1-month price returns
* 3-month price returns
* 6-month price returns
* 1-year price returns

We have used the abbreviation `hqm` often. It stands for `high-quality momentum`.


```python
hqm_columns = [
                'Ticker', 
                'Price', 
                'Number of Shares to Buy', 
                'One-Year Price Return', 
                'One-Year Return Percentile',
                'Six-Month Price Return',
                'Six-Month Return Percentile',
                'Three-Month Price Return',
                'Three-Month Return Percentile',
                'One-Month Price Return',
                'One-Month Return Percentile',
                'HQM Score'
                ]

hqm_dataframe = pd.DataFrame(columns = hqm_columns)

for symbol_string in symbol_strings:
#     print(symbol_strings)
    batch_api_call_url = f'https://sandbox.iexapis.com/stable/stock/market/batch/?types=stats,quote&symbols={symbol_string}&token={IEX_CLOUD_API_TOKEN}'
    data = requests.get(batch_api_call_url).json()
    for symbol in symbol_string.split(','):
        hqm_dataframe = hqm_dataframe.append(
                                        pd.Series([symbol, 
                                                   data[symbol]['quote']['latestPrice'],
                                                   'N/A',
                                                   data[symbol]['stats']['year1ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month6ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month3ChangePercent'],
                                                   'N/A',
                                                   data[symbol]['stats']['month1ChangePercent'],
                                                   'N/A',
                                                   'N/A'
                                                   ], 
                                                  index = hqm_columns), 
                                        ignore_index = True)
        
hqm_dataframe.columns
```




    Index(['Ticker', 'Price', 'Number of Shares to Buy', 'One-Year Price Return',
           'One-Year Return Percentile', 'Six-Month Price Return',
           'Six-Month Return Percentile', 'Three-Month Price Return',
           'Three-Month Return Percentile', 'One-Month Price Return',
           'One-Month Return Percentile', 'HQM Score'],
          dtype='object')



## Calculating Momentum Percentiles

We need to calculate percentile scores for the following metrics for every stock:

* `One-Year Price Return`
* `Six-Month Price Return`
* `Three-Month Price Return`
* `One-Month Price Return`



```python
time_periods = [
                'One-Year',
                'Six-Month',
                'Three-Month',
                'One-Month'
                ]

for row in hqm_dataframe.index:
    for time_period in time_periods:
        hqm_dataframe.loc[row, f'{time_period} Return Percentile'] = stats.percentileofscore(hqm_dataframe[f'{time_period} Price Return'], hqm_dataframe.loc[row, f'{time_period} Price Return'])/100

# Print each percentile score to make sure it was calculated properly
for time_period in time_periods:
    print(hqm_dataframe[f'{time_period} Return Percentile'])

#Print the entire DataFrame    
hqm_dataframe
```

    0       0.885149
    1      0.0237624
    2       0.578218
    3       0.992079
    4        0.89703
             ...    
    500     0.211881
    501     0.457426
    502     0.843564
    503     0.255446
    504     0.772277
    Name: One-Year Return Percentile, Length: 505, dtype: object
    0       0.837624
    1      0.0158416
    2       0.839604
    3       0.968317
    4       0.629703
             ...    
    500     0.405941
    501      0.39604
    502     0.906931
    503     0.227723
    504     0.776238
    Name: Six-Month Return Percentile, Length: 505, dtype: object
    0      0.473267
    1      0.908911
    2      0.643564
    3      0.887129
    4       0.19802
             ...   
    500    0.374257
    501    0.544554
    502    0.611881
    503     0.70297
    504    0.665347
    Name: Three-Month Return Percentile, Length: 505, dtype: object
    0       0.530693
    1       0.827723
    2       0.742574
    3       0.879208
    4      0.0693069
             ...    
    500     0.370297
    501     0.762376
    502     0.641584
    503     0.312871
    504     0.792079
    Name: One-Month Return Percentile, Length: 505, dtype: object





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Ticker</th>
      <th>Price</th>
      <th>Number of Shares to Buy</th>
      <th>One-Year Price Return</th>
      <th>One-Year Return Percentile</th>
      <th>Six-Month Price Return</th>
      <th>Six-Month Return Percentile</th>
      <th>Three-Month Price Return</th>
      <th>Three-Month Return Percentile</th>
      <th>One-Month Price Return</th>
      <th>One-Month Return Percentile</th>
      <th>HQM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>98.19</td>
      <td>N/A</td>
      <td>0.444090</td>
      <td>0.885149</td>
      <td>0.147456</td>
      <td>0.837624</td>
      <td>0.221461</td>
      <td>0.473267</td>
      <td>0.093820</td>
      <td>0.530693</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>13.89</td>
      <td>N/A</td>
      <td>-0.526494</td>
      <td>0.0237624</td>
      <td>-0.564540</td>
      <td>0.0158416</td>
      <td>0.509431</td>
      <td>0.908911</td>
      <td>0.172430</td>
      <td>0.827723</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>161.13</td>
      <td>N/A</td>
      <td>0.088066</td>
      <td>0.578218</td>
      <td>0.148378</td>
      <td>0.839604</td>
      <td>0.295700</td>
      <td>0.643564</td>
      <td>0.144608</td>
      <td>0.742574</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>467.65</td>
      <td>N/A</td>
      <td>1.171724</td>
      <td>0.992079</td>
      <td>0.401695</td>
      <td>0.968317</td>
      <td>0.474900</td>
      <td>0.887129</td>
      <td>0.189840</td>
      <td>0.879208</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>97.29</td>
      <td>N/A</td>
      <td>0.478770</td>
      <td>0.89703</td>
      <td>0.001711</td>
      <td>0.629703</td>
      <td>0.077880</td>
      <td>0.19802</td>
      <td>-0.024533</td>
      <td>0.0693069</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>500</th>
      <td>YUM</td>
      <td>93.51</td>
      <td>N/A</td>
      <td>-0.214524</td>
      <td>0.211881</td>
      <td>-0.116757</td>
      <td>0.405941</td>
      <td>0.161550</td>
      <td>0.374257</td>
      <td>0.068180</td>
      <td>0.370297</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>501</th>
      <td>ZBH</td>
      <td>140.16</td>
      <td>N/A</td>
      <td>0.003007</td>
      <td>0.457426</td>
      <td>-0.127491</td>
      <td>0.39604</td>
      <td>0.250906</td>
      <td>0.544554</td>
      <td>0.151414</td>
      <td>0.762376</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>502</th>
      <td>ZBRA</td>
      <td>285.97</td>
      <td>N/A</td>
      <td>0.373952</td>
      <td>0.843564</td>
      <td>0.223856</td>
      <td>0.906931</td>
      <td>0.283668</td>
      <td>0.611881</td>
      <td>0.115379</td>
      <td>0.641584</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>503</th>
      <td>ZION</td>
      <td>34.95</td>
      <td>N/A</td>
      <td>-0.161814</td>
      <td>0.255446</td>
      <td>-0.255398</td>
      <td>0.227723</td>
      <td>0.328190</td>
      <td>0.70297</td>
      <td>0.055265</td>
      <td>0.312871</td>
      <td>N/A</td>
    </tr>
    <tr>
      <th>504</th>
      <td>ZTS</td>
      <td>163.66</td>
      <td>N/A</td>
      <td>0.290969</td>
      <td>0.772277</td>
      <td>0.102747</td>
      <td>0.776238</td>
      <td>0.307237</td>
      <td>0.665347</td>
      <td>0.157549</td>
      <td>0.792079</td>
      <td>N/A</td>
    </tr>
  </tbody>
</table>
<p>505 rows × 12 columns</p>
</div>



## Calculating the HQM Score

We'll now calculate our `HQM Score`, which we'll use to filter for stocks in this investing strategy.

The `HQM Score` will be the arithmetic mean of the 4 momentum percentile scores that we calculated in the last section.

To calculate arithmetic mean, we will use the `mean` function from Python's built-in `statistics` module.


```python
from statistics import mean

for row in hqm_dataframe.index:
    momentum_percentiles = []
    for time_period in time_periods:
        momentum_percentiles.append(hqm_dataframe.loc[row, f'{time_period} Return Percentile'])
    hqm_dataframe.loc[row, 'HQM Score'] = mean(momentum_percentiles)
```

## Selecting the 50 Best Momentum Stocks

We get the top 50 best performing momentum stock by sorting the DataFrame on the `HQM Score` column in desc order and drop all but top 50 rows.



```python
hqm_dataframe.sort_values(by = 'HQM Score', ascending = False)
hqm_dataframe = hqm_dataframe[:51]
```

## Calculating the Number of Shares to Buy

We'll use the `portfolio_input` function to accept our portfolio size.


```python
portfolio_input()
```

    Enter the value of your portfolio:1000000



```python
position_size = float(portfolio_size) / len(hqm_dataframe.index)
for i in range(0, len(hqm_dataframe['Ticker'])-1):
    hqm_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / hqm_dataframe['Price'][i])
hqm_dataframe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Ticker</th>
      <th>Price</th>
      <th>Number of Shares to Buy</th>
      <th>One-Year Price Return</th>
      <th>One-Year Return Percentile</th>
      <th>Six-Month Price Return</th>
      <th>Six-Month Return Percentile</th>
      <th>Three-Month Price Return</th>
      <th>Three-Month Return Percentile</th>
      <th>One-Month Price Return</th>
      <th>One-Month Return Percentile</th>
      <th>HQM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>98.19</td>
      <td>199</td>
      <td>0.444090</td>
      <td>0.885149</td>
      <td>0.147456</td>
      <td>0.837624</td>
      <td>0.221461</td>
      <td>0.473267</td>
      <td>0.093820</td>
      <td>0.530693</td>
      <td>0.681683</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AAL</td>
      <td>13.89</td>
      <td>1411</td>
      <td>-0.526494</td>
      <td>0.0237624</td>
      <td>-0.564540</td>
      <td>0.0158416</td>
      <td>0.509431</td>
      <td>0.908911</td>
      <td>0.172430</td>
      <td>0.827723</td>
      <td>0.444059</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AAP</td>
      <td>161.13</td>
      <td>121</td>
      <td>0.088066</td>
      <td>0.578218</td>
      <td>0.148378</td>
      <td>0.839604</td>
      <td>0.295700</td>
      <td>0.643564</td>
      <td>0.144608</td>
      <td>0.742574</td>
      <td>0.70099</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AAPL</td>
      <td>467.65</td>
      <td>41</td>
      <td>1.171724</td>
      <td>0.992079</td>
      <td>0.401695</td>
      <td>0.968317</td>
      <td>0.474900</td>
      <td>0.887129</td>
      <td>0.189840</td>
      <td>0.879208</td>
      <td>0.931683</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABBV</td>
      <td>97.29</td>
      <td>201</td>
      <td>0.478770</td>
      <td>0.89703</td>
      <td>0.001711</td>
      <td>0.629703</td>
      <td>0.077880</td>
      <td>0.19802</td>
      <td>-0.024533</td>
      <td>0.0693069</td>
      <td>0.448515</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ABC</td>
      <td>104.97</td>
      <td>186</td>
      <td>0.163705</td>
      <td>0.651485</td>
      <td>0.100112</td>
      <td>0.774257</td>
      <td>0.241867</td>
      <td>0.522772</td>
      <td>0.066990</td>
      <td>0.364356</td>
      <td>0.578218</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ABMD</td>
      <td>305.78</td>
      <td>64</td>
      <td>0.546138</td>
      <td>0.942574</td>
      <td>0.874783</td>
      <td>0.99802</td>
      <td>0.645795</td>
      <td>0.954455</td>
      <td>0.145153</td>
      <td>0.746535</td>
      <td>0.910396</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ABT</td>
      <td>102.94</td>
      <td>190</td>
      <td>0.161073</td>
      <td>0.647525</td>
      <td>0.139130</td>
      <td>0.823762</td>
      <td>0.093917</td>
      <td>0.233663</td>
      <td>0.081815</td>
      <td>0.457426</td>
      <td>0.540594</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ACN</td>
      <td>237.88</td>
      <td>82</td>
      <td>0.197484</td>
      <td>0.69505</td>
      <td>0.085519</td>
      <td>0.762376</td>
      <td>0.279570</td>
      <td>0.605941</td>
      <td>0.067045</td>
      <td>0.366337</td>
      <td>0.607426</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ADBE</td>
      <td>453.91</td>
      <td>43</td>
      <td>0.532839</td>
      <td>0.934653</td>
      <td>0.196705</td>
      <td>0.879208</td>
      <td>0.252382</td>
      <td>0.550495</td>
      <td>0.006829</td>
      <td>0.150495</td>
      <td>0.628713</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ADI</td>
      <td>120.52</td>
      <td>162</td>
      <td>0.056525</td>
      <td>0.522772</td>
      <td>0.002920</td>
      <td>0.631683</td>
      <td>0.149173</td>
      <td>0.360396</td>
      <td>0.017244</td>
      <td>0.174257</td>
      <td>0.422277</td>
    </tr>
    <tr>
      <th>11</th>
      <td>ADM</td>
      <td>46.62</td>
      <td>420</td>
      <td>0.178916</td>
      <td>0.679208</td>
      <td>-0.018869</td>
      <td>0.580198</td>
      <td>0.328413</td>
      <td>0.70495</td>
      <td>0.123837</td>
      <td>0.687129</td>
      <td>0.662871</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ADP</td>
      <td>139.82</td>
      <td>140</td>
      <td>-0.174896</td>
      <td>0.243564</td>
      <td>-0.227657</td>
      <td>0.265347</td>
      <td>0.037317</td>
      <td>0.134653</td>
      <td>-0.042030</td>
      <td>0.0435644</td>
      <td>0.171782</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ADSK</td>
      <td>234.20</td>
      <td>83</td>
      <td>0.528696</td>
      <td>0.928713</td>
      <td>0.116990</td>
      <td>0.794059</td>
      <td>0.312257</td>
      <td>0.677228</td>
      <td>-0.001259</td>
      <td>0.122772</td>
      <td>0.630693</td>
    </tr>
    <tr>
      <th>14</th>
      <td>AEE</td>
      <td>85.23</td>
      <td>230</td>
      <td>0.073499</td>
      <td>0.556436</td>
      <td>-0.044706</td>
      <td>0.538614</td>
      <td>0.202938</td>
      <td>0.447525</td>
      <td>0.086675</td>
      <td>0.483168</td>
      <td>0.506436</td>
    </tr>
    <tr>
      <th>15</th>
      <td>AEP</td>
      <td>87.88</td>
      <td>223</td>
      <td>-0.069760</td>
      <td>0.368317</td>
      <td>-0.186159</td>
      <td>0.326733</td>
      <td>0.084255</td>
      <td>0.217822</td>
      <td>-0.007456</td>
      <td>0.106931</td>
      <td>0.25495</td>
    </tr>
    <tr>
      <th>16</th>
      <td>AES</td>
      <td>17.54</td>
      <td>1117</td>
      <td>0.141966</td>
      <td>0.635644</td>
      <td>-0.177281</td>
      <td>0.336634</td>
      <td>0.510037</td>
      <td>0.910891</td>
      <td>0.186968</td>
      <td>0.873267</td>
      <td>0.689109</td>
    </tr>
    <tr>
      <th>17</th>
      <td>AFL</td>
      <td>39.27</td>
      <td>499</td>
      <td>-0.298263</td>
      <td>0.116832</td>
      <td>-0.283044</td>
      <td>0.190099</td>
      <td>0.178662</td>
      <td>0.4</td>
      <td>0.082055</td>
      <td>0.459406</td>
      <td>0.291584</td>
    </tr>
    <tr>
      <th>18</th>
      <td>AIG</td>
      <td>31.76</td>
      <td>617</td>
      <td>-0.468456</td>
      <td>0.0356436</td>
      <td>-0.398200</td>
      <td>0.0673267</td>
      <td>0.242767</td>
      <td>0.528713</td>
      <td>0.042150</td>
      <td>0.271287</td>
      <td>0.225743</td>
    </tr>
    <tr>
      <th>19</th>
      <td>AIV</td>
      <td>39.00</td>
      <td>502</td>
      <td>-0.260956</td>
      <td>0.168317</td>
      <td>-0.323003</td>
      <td>0.142574</td>
      <td>0.096556</td>
      <td>0.243564</td>
      <td>0.004109</td>
      <td>0.140594</td>
      <td>0.173762</td>
    </tr>
    <tr>
      <th>20</th>
      <td>AIZ</td>
      <td>129.93</td>
      <td>150</td>
      <td>0.016575</td>
      <td>0.475248</td>
      <td>-0.116835</td>
      <td>0.40396</td>
      <td>0.418900</td>
      <td>0.839604</td>
      <td>0.254327</td>
      <td>0.948515</td>
      <td>0.666832</td>
    </tr>
    <tr>
      <th>21</th>
      <td>AJG</td>
      <td>111.00</td>
      <td>176</td>
      <td>0.195680</td>
      <td>0.689109</td>
      <td>-0.011750</td>
      <td>0.6</td>
      <td>0.253483</td>
      <td>0.554455</td>
      <td>0.092902</td>
      <td>0.520792</td>
      <td>0.591089</td>
    </tr>
    <tr>
      <th>22</th>
      <td>AKAM</td>
      <td>110.00</td>
      <td>178</td>
      <td>0.206310</td>
      <td>0.70495</td>
      <td>0.075705</td>
      <td>0.752475</td>
      <td>0.125834</td>
      <td>0.312871</td>
      <td>-0.027015</td>
      <td>0.0594059</td>
      <td>0.457426</td>
    </tr>
    <tr>
      <th>23</th>
      <td>ALB</td>
      <td>94.85</td>
      <td>206</td>
      <td>0.339750</td>
      <td>0.823762</td>
      <td>0.036265</td>
      <td>0.691089</td>
      <td>0.528245</td>
      <td>0.918812</td>
      <td>0.120477</td>
      <td>0.673267</td>
      <td>0.776733</td>
    </tr>
    <tr>
      <th>24</th>
      <td>ALGN</td>
      <td>309.25</td>
      <td>63</td>
      <td>0.692338</td>
      <td>0.970297</td>
      <td>0.131757</td>
      <td>0.815842</td>
      <td>0.554489</td>
      <td>0.928713</td>
      <td>0.143933</td>
      <td>0.738614</td>
      <td>0.863366</td>
    </tr>
    <tr>
      <th>25</th>
      <td>ALK</td>
      <td>38.63</td>
      <td>507</td>
      <td>-0.387440</td>
      <td>0.0633663</td>
      <td>-0.438559</td>
      <td>0.0534653</td>
      <td>0.519770</td>
      <td>0.916832</td>
      <td>0.103833</td>
      <td>0.576238</td>
      <td>0.402475</td>
    </tr>
    <tr>
      <th>26</th>
      <td>ALL</td>
      <td>99.53</td>
      <td>197</td>
      <td>-0.079870</td>
      <td>0.354455</td>
      <td>-0.230788</td>
      <td>0.261386</td>
      <td>0.045750</td>
      <td>0.146535</td>
      <td>0.088925</td>
      <td>0.49703</td>
      <td>0.314851</td>
    </tr>
    <tr>
      <th>27</th>
      <td>ALLE</td>
      <td>105.13</td>
      <td>186</td>
      <td>0.071934</td>
      <td>0.542574</td>
      <td>-0.246470</td>
      <td>0.241584</td>
      <td>0.112362</td>
      <td>0.277228</td>
      <td>0.024073</td>
      <td>0.20396</td>
      <td>0.316337</td>
    </tr>
    <tr>
      <th>28</th>
      <td>ALXN</td>
      <td>107.60</td>
      <td>182</td>
      <td>-0.079570</td>
      <td>0.356436</td>
      <td>-0.006381</td>
      <td>0.60396</td>
      <td>0.003280</td>
      <td>0.0792079</td>
      <td>-0.040910</td>
      <td>0.0455446</td>
      <td>0.271287</td>
    </tr>
    <tr>
      <th>29</th>
      <td>AMAT</td>
      <td>68.10</td>
      <td>287</td>
      <td>0.389577</td>
      <td>0.855446</td>
      <td>-0.013556</td>
      <td>0.594059</td>
      <td>0.301391</td>
      <td>0.651485</td>
      <td>0.082413</td>
      <td>0.461386</td>
      <td>0.640594</td>
    </tr>
    <tr>
      <th>30</th>
      <td>AMCR</td>
      <td>11.52</td>
      <td>1702</td>
      <td>0.073093</td>
      <td>0.552475</td>
      <td>0.105582</td>
      <td>0.778218</td>
      <td>0.238793</td>
      <td>0.510891</td>
      <td>0.077579</td>
      <td>0.433663</td>
      <td>0.568812</td>
    </tr>
    <tr>
      <th>31</th>
      <td>AMD</td>
      <td>85.23</td>
      <td>230</td>
      <td>1.590065</td>
      <td>0.99604</td>
      <td>0.532176</td>
      <td>0.984158</td>
      <td>0.597011</td>
      <td>0.948515</td>
      <td>0.560304</td>
      <td>0.99802</td>
      <td>0.981683</td>
    </tr>
    <tr>
      <th>32</th>
      <td>AME</td>
      <td>104.38</td>
      <td>187</td>
      <td>0.177719</td>
      <td>0.677228</td>
      <td>-0.002080</td>
      <td>0.615842</td>
      <td>0.305470</td>
      <td>0.659406</td>
      <td>0.147190</td>
      <td>0.752475</td>
      <td>0.676238</td>
    </tr>
    <tr>
      <th>33</th>
      <td>AMGN</td>
      <td>253.49</td>
      <td>77</td>
      <td>0.176726</td>
      <td>0.671287</td>
      <td>0.086140</td>
      <td>0.764356</td>
      <td>0.020895</td>
      <td>0.112871</td>
      <td>-0.034769</td>
      <td>0.0534653</td>
      <td>0.400495</td>
    </tr>
    <tr>
      <th>34</th>
      <td>AMP</td>
      <td>166.96</td>
      <td>117</td>
      <td>0.254633</td>
      <td>0.736634</td>
      <td>-0.101617</td>
      <td>0.429703</td>
      <td>0.385995</td>
      <td>0.80198</td>
      <td>0.089628</td>
      <td>0.50099</td>
      <td>0.617327</td>
    </tr>
    <tr>
      <th>35</th>
      <td>AMT</td>
      <td>252.70</td>
      <td>77</td>
      <td>0.137500</td>
      <td>0.631683</td>
      <td>-0.022213</td>
      <td>0.574257</td>
      <td>0.091099</td>
      <td>0.231683</td>
      <td>-0.018941</td>
      <td>0.0871287</td>
      <td>0.381188</td>
    </tr>
    <tr>
      <th>36</th>
      <td>AMZN</td>
      <td>3267.82</td>
      <td>6</td>
      <td>0.753428</td>
      <td>0.978218</td>
      <td>0.471720</td>
      <td>0.974257</td>
      <td>0.339324</td>
      <td>0.728713</td>
      <td>0.018781</td>
      <td>0.178218</td>
      <td>0.714851</td>
    </tr>
    <tr>
      <th>37</th>
      <td>ANET</td>
      <td>225.98</td>
      <td>86</td>
      <td>-0.065031</td>
      <td>0.374257</td>
      <td>-0.093866</td>
      <td>0.449505</td>
      <td>0.011798</td>
      <td>0.0910891</td>
      <td>0.024541</td>
      <td>0.207921</td>
      <td>0.280693</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ANSS</td>
      <td>321.00</td>
      <td>61</td>
      <td>0.451591</td>
      <td>0.887129</td>
      <td>0.058949</td>
      <td>0.722772</td>
      <td>0.229415</td>
      <td>0.487129</td>
      <td>0.044242</td>
      <td>0.283168</td>
      <td>0.59505</td>
    </tr>
    <tr>
      <th>39</th>
      <td>ANTM</td>
      <td>293.84</td>
      <td>66</td>
      <td>-0.013031</td>
      <td>0.439604</td>
      <td>-0.053364</td>
      <td>0.514851</td>
      <td>0.058803</td>
      <td>0.164356</td>
      <td>0.104273</td>
      <td>0.582178</td>
      <td>0.425248</td>
    </tr>
    <tr>
      <th>40</th>
      <td>AON</td>
      <td>196.84</td>
      <td>99</td>
      <td>0.006102</td>
      <td>0.463366</td>
      <td>-0.183042</td>
      <td>0.332673</td>
      <td>0.010224</td>
      <td>0.0891089</td>
      <td>-0.026247</td>
      <td>0.0633663</td>
      <td>0.237129</td>
    </tr>
    <tr>
      <th>41</th>
      <td>AOS</td>
      <td>50.31</td>
      <td>389</td>
      <td>0.101690</td>
      <td>0.59802</td>
      <td>0.141115</td>
      <td>0.825743</td>
      <td>0.260257</td>
      <td>0.564356</td>
      <td>0.032405</td>
      <td>0.231683</td>
      <td>0.55495</td>
    </tr>
    <tr>
      <th>42</th>
      <td>APA</td>
      <td>16.40</td>
      <td>1195</td>
      <td>-0.291451</td>
      <td>0.124752</td>
      <td>-0.441406</td>
      <td>0.0514851</td>
      <td>0.485613</td>
      <td>0.893069</td>
      <td>0.260429</td>
      <td>0.952475</td>
      <td>0.505446</td>
    </tr>
    <tr>
      <th>43</th>
      <td>APD</td>
      <td>295.10</td>
      <td>66</td>
      <td>0.242925</td>
      <td>0.728713</td>
      <td>0.124499</td>
      <td>0.807921</td>
      <td>0.273219</td>
      <td>0.592079</td>
      <td>0.040389</td>
      <td>0.265347</td>
      <td>0.598515</td>
    </tr>
    <tr>
      <th>44</th>
      <td>APH</td>
      <td>110.83</td>
      <td>176</td>
      <td>0.255236</td>
      <td>0.738614</td>
      <td>0.053364</td>
      <td>0.714851</td>
      <td>0.342563</td>
      <td>0.740594</td>
      <td>0.168114</td>
      <td>0.819802</td>
      <td>0.753465</td>
    </tr>
    <tr>
      <th>45</th>
      <td>APTV</td>
      <td>91.40</td>
      <td>214</td>
      <td>0.072118</td>
      <td>0.544554</td>
      <td>-0.011731</td>
      <td>0.60198</td>
      <td>0.490744</td>
      <td>0.89505</td>
      <td>0.172564</td>
      <td>0.829703</td>
      <td>0.717822</td>
    </tr>
    <tr>
      <th>46</th>
      <td>ARE</td>
      <td>174.20</td>
      <td>112</td>
      <td>0.208731</td>
      <td>0.706931</td>
      <td>-0.001613</td>
      <td>0.619802</td>
      <td>0.214849</td>
      <td>0.465347</td>
      <td>0.078630</td>
      <td>0.441584</td>
      <td>0.558416</td>
    </tr>
    <tr>
      <th>47</th>
      <td>ATO</td>
      <td>108.98</td>
      <td>179</td>
      <td>-0.039357</td>
      <td>0.415842</td>
      <td>-0.125705</td>
      <td>0.4</td>
      <td>0.089492</td>
      <td>0.229703</td>
      <td>0.053203</td>
      <td>0.306931</td>
      <td>0.338119</td>
    </tr>
    <tr>
      <th>48</th>
      <td>ATVI</td>
      <td>82.88</td>
      <td>236</td>
      <td>0.706749</td>
      <td>0.976238</td>
      <td>0.310780</td>
      <td>0.950495</td>
      <td>0.110910</td>
      <td>0.273267</td>
      <td>0.048812</td>
      <td>0.289109</td>
      <td>0.622277</td>
    </tr>
    <tr>
      <th>49</th>
      <td>AVB</td>
      <td>156.62</td>
      <td>125</td>
      <td>-0.263600</td>
      <td>0.162376</td>
      <td>-0.336301</td>
      <td>0.120792</td>
      <td>0.012586</td>
      <td>0.0930693</td>
      <td>0.005162</td>
      <td>0.142574</td>
      <td>0.129703</td>
    </tr>
    <tr>
      <th>50</th>
      <td>AVGO</td>
      <td>336.73</td>
      <td>N/A</td>
      <td>0.177460</td>
      <td>0.675248</td>
      <td>0.030386</td>
      <td>0.679208</td>
      <td>0.278870</td>
      <td>0.60198</td>
      <td>0.074210</td>
      <td>0.405941</td>
      <td>0.590594</td>
    </tr>
  </tbody>
</table>
</div>



## Formatting Our Excel Output

We will be using the XlsxWriter library for Python to create nicely-formatted Excel files.



```python
writer = pd.ExcelWriter('momentum_strategy.xlsx', engine='xlsxwriter')
hqm_dataframe.to_excel(writer, sheet_name='Momentum Strategy', index = False)
```

## Creating the Formats We'll Need For Our .xlsx File



```python
background_color = '#0a0a23'
font_color = '#ffffff'

string_template = writer.book.add_format(
        {
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

dollar_template = writer.book.add_format(
        {
            'num_format':'$0.00',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

integer_template = writer.book.add_format(
        {
            'num_format':'0',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )

percent_template = writer.book.add_format(
        {
            'num_format':'0.0%',
            'font_color': font_color,
            'bg_color': background_color,
            'border': 1
        }
    )
```


```python
column_formats = { 
                    'A': ['Ticker', string_template],
                    'B': ['Price', dollar_template],
                    'C': ['Number of Shares to Buy', integer_template],
                    'D': ['One-Year Price Return', percent_template],
                    'E': ['One-Year Return Percentile', percent_template],
                    'F': ['Six-Month Price Return', percent_template],
                    'G': ['Six-Month Return Percentile', percent_template],
                    'H': ['Three-Month Price Return', percent_template],
                    'I': ['Three-Month Return Percentile', percent_template],
                    'J': ['One-Month Price Return', percent_template],
                    'K': ['One-Month Return Percentile', percent_template],
                    'L': ['HQM Score', integer_template]
                    }

for column in column_formats.keys():
    writer.sheets['Momentum Strategy'].set_column(f'{column}:{column}', 20, column_formats[column][1])
    writer.sheets['Momentum Strategy'].write(f'{column}1', column_formats[column][0], string_template)
```

## Saving Our Excel Output



```python
writer.save()
```
