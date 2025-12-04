# Building a Dashboard for Crypto Bros (Part 1)

[Home](https://joncwmusic.github.io/Data-Blog-Site/)

So, as you may know crypto currency, web 3.0, and NFTs have been circulating in the mainstream for quite a while 
as the last hope for the everyday person to accumulate riches on the level of the wealthiest individuals in the world. 
And while it may be weird that the cashier at best buy has a really strong opinion about monetary policy and centralized currency 
it’s at least worth it to monitor where these things are going.
And to that end I’m going to join the tokenized hype in the least active way possible by turning it into a blog post and tutorial.

To be clear:

**I am not a credentialed expert in finance. I know nothing about to create blockchain tokens, meme coins, or cryptocurrencies.
This is a project for exploration and learning, not a serious attempt at an in depth analysis.**

## The Project

I wanted to set up a dashboard monitoring crypto price action metrics at the daily level.
Nothing real-time. Ain’t nobody got the budget for that.
But I also wanted to illustrate the challenges and trade offs of setting something like that up for personal use instead of throwing money at coinbase 
(ya’ll shouldn’t have rejected my resume Mr. or Mrs. `no-reply@coinbase.recruiting.com`). 

And with the ramblings of a man stung by rejection, a personal project was born. 
A personal project designed to convince crypto bros they can have valuable insights that can beat the crypto market
and win one over on the massive institutions that have the resources and people power to bankrupt you just because they can.

So the plan is simple:

- Source the data from somewhere
- Store the data and automate uploads
- ?
- Profit

As it turns out, this is the best practice for most data teams if you want to justify a data-based initiative without any real business impact.
But don’t worry, I’m sure your team is getting exactly the data it needs to solve problems for the business application in a streamlined and synergistic fashion, right? 
Let’s circle back on that later.

## Getting Data
Okay so I want data and I need to get data. How get data? 

I could arduously nab the prices of each token day by day and insert the values manually into a table… 
OR I could use my brain, iteratively developed by the darwinistic survival based critial thinking that allowed
humanity to survive long enough to develop electronic addiction machines in your pocket, and find an automated solution.
In fact, there are several ways to get that data without leaving your chair and that would be through various APIs.
In this project, I used coingecko as it was free. Feel free to peruse their documentation here if you’re crazy enough to try gambling on meme coins 
https://docs.coingecko.com/

```Python
# Python Code

# import Libraries
import requests
import pandas as pd
from datetime import datetime

def fetch_historical_prices(coin, currency):
    """
    :param coin: string for token name
    :param currency: the currency to convert to. Default usd
    :return: dataframes with 90 day historicals of price, volume, and market cap
    """
    
    # coin gecko let's you nab price, marketcap, and trading volume on the daily level.
    price_records = []
    market_cap_records = []
    volumes_records = []

    # endpoint for each coin
    url = f"https://api.coingecko.com/api/v3/coins/{coin}/market_chart"
    params = {'vs_currency': currency, 'days': 90, 'interval': 'daily', 'precision': 3}
    response = requests.get(url, params=params)
    data = response.json()

    #extract the componenets from the json file to get the relevent data
    for entry in data['prices']:
        price_records.append({
            'token': coin,
            'date': datetime.utcfromtimestamp(entry[0] / 1000).date(),
            'price_' + currency: entry[1]
        })

    for entry in data['market_caps']:
        market_cap_records.append({
            'token': coin,
            'date': datetime.utcfromtimestamp(entry[0] / 1000).date(),
            'market_cap_' + currency: entry[1]
        })

    for entry in data['total_volumes']:
        volumes_records.append({
            'token': coin,
            'date': datetime.utcfromtimestamp(entry[0] / 1000).date(),
            'volume': entry[1]
        })

    # create the dataframes to be output
    df_prices = pd.DataFrame(price_records)
    df_market_cap = pd.DataFrame(market_cap_records)
    df_volumes = pd.DataFrame(volumes_records)

    return df_prices, df_market_cap, df_volumes

```

As you can see from the above script, there's nothing really special going on. I ask coingecko "Hey, can I get that daily market data" and coingecko goes "Sure, no problem, bud. Just don't ask too much or I'll have to start charging you and breaking your application."
API rate limits are no joke so always be paying attention. If you're not interested in crypto and want to be a traditional finance person,
AlphaVantage is an API with a free tier where you can only make about 25 requests per day. Otherwise you gotta pay up.
Find that here: https://www.alphavantage.co/documentation/

So now the data lives in a dataframe in our application but if we don't want to keep the garbage collector busy, we might want to put this data somewhere.

## Storing data

So if you're going to store this, I don't recommend saving or exporting your dataframes to csv's and permanently reliquishing your data to comma seperated data engineering hell. Instead, you should probably set up a local sqlite instance (you can also use this to cache your data so you don't need to make as many requests to the coingecko API especially if you're calling it for several different tokens).

Here's a script to store the dataframe into a sqlite database:

```Python
# Python code

import pandas as pd
import sqlite3

data = {
    "date": pd.date_range("2025-01-01", periods=5),
    "value": [10, 20, 15, 25, 30],
}
df = pd.DataFrame(data)

# Create or connect to a local SQLite database file
db_path = "example.db"
conn = sqlite3.connect(db_path)

# to_sql set to replace but you should append and remove duplicates if you want to cumulatively collect data
table_name = "crypto_data_historicals"
df.to_sql(table_name, conn, if_exists="replace", index=False)

# Close the connection
conn.close()
```

[Back to Home](https://joncwmusic.github.io/Data-Blog-Site/)





