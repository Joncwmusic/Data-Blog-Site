# Building a Dashboard for Crypto Bros (Part 1)

[Home](https://joncwmusic.github.io/Data-Blog-Site/)

So, as you may know crypto currency, web 3.0, and NFTs have been circulating in the mainstream for quite a while 
as the last hope for the everyday person to accumulate riches on the level of the wealthiest individuals in the world. 
And while it may be weird that the cashier at best buy has a really strong opinion about monetary policy and centralized currency 
it’s at least worth it to monitor where these things are going.
And to that end I’m going to join the tokenized hype in the least active way possible by turning it into a blog post and tutorial.

To be clear:

**I am not a credentialed expert in finance. I know nothing about how to create blockchain tokens, meme coins, or cryptocurrencies.
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

```python
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

## Storing data (Locally)

So if you're going to store this, I don't recommend saving or exporting your dataframes to csv's and permanently reliquishing your data to comma seperated data engineering hell. Instead, you should probably set up a local sqlite instance (you can also use this to cache your data so you don't need to make as many requests to the coingecko API especially if you're calling it for several different tokens).

Here's a script to store the dataframe into a sqlite database:

```python
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

## Storing Data (BigQuery)

Okay maybe you're good on toy projects and instead want to a bit more enterprising.
Something like a big data, cloud based solution for storage and downstream analytics.
You have a couple options from using S3 or redshift through AWS, or hosting a SQLServer instance through microsoft or even google.
For this project, I used BigQuery because that's what I'm familiar with and honestly for a project at this scale the choice is up to you.

When using BigQuery you need to make sure that your application (the scripts you're writing) has permissions to upload and edit tables in bigquery.
You accomplish this by adding a service account which is like user permission but for the application you're making.
You then use the credentials from the service account to let google know your application can safely ruin your data infrastructure.

Luckily python already has libraries dedicated to connecting to your cloud based project and shooting up your dataframes into the internet ether.
I made a couple of functions along with the upload for some sanity checks starting with a function called "table_exists".


```python
# Python Code

from dotenv import load_dotenv
from google.cloud import bigquery as bq
import os

# environment variable should be stored in .env
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "filepath/google_credential_proejct_file.json"
load_dotenv()

# check if the table even exists. Useful for uploading and determining insert logic.
def table_exists(table_id):
    client = bq.Client()
    try:
        client.get_table(table_id)
        print('Found table', table_id)
        return True
    except AttributeError as e:
        print('Unable to find table with table_id: ' , table_id)
        return False
```

Another useful function to have if you're going to upload directly to bigQuery is something like a get_num_rows function.
Calling this before and after an upload will give you an idea of how much data was imported or more importantly,
no change would signal nothing probably happened and there may be a bug you have to search for.

```python
# Python Code

# send a query to ge the number of rows from a table.
def get_num_rows(table_id, where_clause = '1=1'):
    client = bq.Client()
    query  = f"""
    SELECT COUNT(1) AS num_rows FROM `{table_id}` WHERE {where_clause}
    """
    val = client.query(query).to_dataframe()
    return val.iloc[0][0]
```

Now let's get into the meat and potatoes of the upload. The upload itself is effectively a big BigQuery query to send a dataframe up as a temp table, and then send a query to merge, append, or replace the new table with the temp table.

In code:

```python
# Python Code

# select the dataframe from your project, set the mode which is defaulted to append, and determine merge keys for when mode=merge
def upload_to_bigquery(df, table_id, mode = "append", merge_keys = None):
    table_exists(table_id)
    client = bq.Client()

    # catch when mode is invalid
    if mode not in {"append", "replace", "create", "merge"}:
        raise ValueError("Invalid mode. Use one of: append, replace, create, merge")

    # case when the table doesn't exist and needs to be created
    if mode == "create" and not table_exists(table_id):
        job = client.load_table_from_dataframe(df, table_id, job_config=bq.LoadJobConfig(autodetect=True))
        job.result()
        print(f"Created and loaded table {table_id} with {len(df)} rows.")
        return

    # case when the table exists and just needs the extra data
    if mode == "append":
        job_config = bq.LoadJobConfig(write_disposition="WRITE_APPEND", autodetect=True)
        job = client.load_table_from_dataframe(df, table_id, job_config=job_config)
        job.result()
        print(f"Appended {len(df)} rows to {table_id}.")
        return

    # case for burning everything down and starting again when you realize you've been uploadign market cap to price and vice versa
    if mode == "replace":
        job_config = bq.LoadJobConfig(write_disposition="WRITE_TRUNCATE", autodetect=True)
        job = client.load_table_from_dataframe(df, table_id, job_config=job_config)
        job.result()
        print(f"Replaced data in {table_id} with {len(df)} rows.")
        return

    # the likely case for cumulative data collection to avoid duplicates
    if mode == "merge":
        if not merge_keys:
            raise ValueError("Merge key must be provided for merge mode")

        temp_table_id = table_id + "_temp"
        job = client.load_table_from_dataframe(df, temp_table_id, job_config=bq.LoadJobConfig(
            write_disposition="WRITE_TRUNCATE", autodetect=True
        ))
        job.result()

        project_id, dataset_id, table_name = table_id.split(".")

        on_clause = " AND ".join([f"T.{k} = S.{k}" for k in merge_keys])

        # formatted query string to merge the temp table into the original table
        merge_sql = f"""
        MERGE `{table_id}` T
        USING `{temp_table_id}` S
        ON {on_clause}       
        WHEN MATCHED THEN
          UPDATE SET {', '.join([f'T.{col} = S.{col}' for col in df.columns if col not in merge_keys])}
        WHEN NOT MATCHED THEN
          INSERT ({', '.join(df.columns)}) 
          VALUES ({', '.join(['S.' + col for col in df.columns])})
        """

        query_job = client.query(merge_sql)
        query_job.result()
        print(f"Merged {len(df)} rows into {table_id} on `{merge_keys}`.")
        client.delete_table(temp_table_id, not_found_ok=True)
```


## To be Continued

A project like this is pretty straightforward for most seasoned career data analysts and engineers.
This project is really more a little "black pepper might be too spicy" as opposed to "level 5 thai-hot spicy noodles" that you'll see in most production environments.
In the next part I'll walk through how I automated everything with the cloud scheduler and cloud run services to get this data uploading weekly.

[Back to Home](https://joncwmusic.github.io/Data-Blog-Site/)





