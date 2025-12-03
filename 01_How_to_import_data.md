# How to Import Data Sources Into Your Project

So you've survived another round of layoffs at the big tech company and you've been cruising at the bare minimum for a while now. 
Instead of improving your skils or working on your fundamentals, you decided to watch videos about improving your skills trying to muster the motivation to change.

You're doing great.

Except, you're KPIs put you in the 89th percentile of staff and the bottom 10% were just given opportunities that aren't within the company.

Ouch.

Now you have to get yourself back to at least the middle of the pack so upper management doesn't catch on to how much you've been slacking.
Your first task is to just get some preliminary data. Your boss doesn't know anything about the data on your internal servers, cloud storage, or even the data in the csvs he has saved locally.
He thinks API means Asian/Pacific Islander because he spent 5 years in HR before becoming a tech hiring manager and eventually, your direct report.
He now makes triple what you make, but you're still the one on the chopping block for having the least number of queries per hour on the server. Because that's a valid way to measure performance *(that was sarcasm)*

## Learning Goals

The goal is simply to lay out the basic options you have to gather data so you can analyze and predict to your hearts content. That way your boss can give you high marks on your performance review that still mean absolutely nothing.

You'll learn how to:

- Import data from public datasets online
- Scrape data from the web with beautiful soup
- Get data via an API
- Use pandas to read in csv files
- Source data directly from a SQLite Database
- Get data from a cloud based data source

With enough time, energy, effort, and caffiene almost anything is possible. Now let's get into it.

### Public Datasets

Public Datasets are easy to work with. You just need a direct url to the dataset and there are several sites whose sole purpose is to give people data. That's great for you, the newly inspired overachiever that's still underachieving. It gives you a way to get your reps in and all you have to do is copy paste like the majority of your career has been so far and will continue to be, even as you copy paste from the LLM you used for your last report.

Data.gov is great to get you started

```python
# Python Code:
import pandas as pd

dataset_url_gov = 'https://data.ny.gov/api/views/e2u6-bmnn/rows.csv'

gov_df = pd.read_csv(dataset_url_gov)
gov_df.head()
```
And now you have data that you can analyse to your hearts content.

### Web Scraping

Web scraping can be very messy but could also be the way you collect some insightful info when you don't have direct access to a site's data in a nice downloadable format or with an API. You can take tables directly from sites, or grab the same piece of info from different webpages. Sometimes you'll even find a meaty `JSON` file with a ton of metadata. Really, it depends on the site and your patience. It's honestly perfect for some competitive intelligence gathering if you know exactly what you're looking for.

Let's go through a couple basic examples.

```python
# Python Code
import requests
from bs4 import BeautifulSoup

url = "https://www.imdb.com/chart/top"
soup = BeautifulSoup(requests.get(url).text, "html.parser")
titles = [tag.text for tag in soup.select(".titleColumn a")]
titles
```

this code block goes to the top imdb movies and gets their titles.

### API

Okay so API stands for application programming interface. It's an interface for your code to work with an application programmatically... let me explain.
Let's say you want to upload videos to a YouTube Channel. But hitting the upload button is just too much work for your overworked fingers that have been typing out that really long reddit thread while you were distracted. You can write a program with your exhausted fingers to do that. But instead of trying to mimic the behavior of a user using a web driver, you can instead use the *Application Programming Interface* that YouTube provides to do it automagically. (If you're actually curious here's a link: [Uploading a YouTube Video via API tutorial](https://developers.google.com/youtube/v3/guides/uploading_a_video).

APIs are useful for automating things but only if the application HAS an API to begin with (Ideally with public documentation). Otherwise you'll have to default to scraping the old fashioned way.

Let's go through this step by step:

- Step one: is to determine what data you want
- Step two: is to tell yourself you can do this even if your title isn't data engineer
- Step three: is to stare at the documentation to discover you CAN get that information with their API (otherwise go back to step 1)
- Step four: request the data with the url string provided by the documentation
- Step five: Convert the data (usually `JSON`) into a usable format
- Bonus Step: Create an analysis that will impress your manager so he can posture to his manager

That last step is how you actually *keep* your job. But let's look at some code

```python
# Python Code

ALPHA_VANTAGE_API_KEY = "demo"
import requests

# You will have to get an API key from alpha vantage directly
url = f"https://www.alphavantage.co/query"
params = {
        "function": "TIME_SERIES_DAILY",
        "symbol": "IBM",
        "apikey": ALPHA_VANTAGE_API_KEY
    }
response = requests.get(url, params=params)
data = response.json()
```
You'll notice the API request is based on three components:

**1 - The Endpoint**

The endpoint is the URL that your application uses to access the data on the server side. In our case we're using alphavantage's API and their query endpoint

**2 - The Parameters**

The parameters are the extra information the server needs to deliver the specific data the client is requesting. In this case the client is asking for daily time series data for the stock with ticker IBM. Notice there's also this parameter *apikey* which is a key given to you by alpha vantage to actually access their endpoint. Do not share these with anyone unless you're not the one footing the bill on rate limits. Then go nuts.

**3 - The Response**

Of course everything up to this point is just to get the client to request the data. The response is what will have the data. And you'll need to put that somewhere. Usually the response comes in a nice `JSON` file and then you can clean it up, put it into a DB or dataframe, and then wait 3 months for everything to crash because the `JSON` formatting changed all of the sudden with new API changes. The data may also come in XML or text formatting so your results may vary.

### Spreadsheets

If you're in this section you either work for a company worn down by the prehistoric hellscape of using excel working under a manager who says VBA is everything you need. Or you work in consulting trying to transition these dinosaurs to the modern day. Either way, you're paying the price for someone else's sin and you have to add another layer to the convoluted engineering pipeline.

No matter how you got here, you can still find some programmatic ways to deal with importing your excel data into a python script so you can mangle it into something that will go into a proper database (that will once again be exported into an excel document to make a chart to go into a powerpoint).

Let's get into our example:

Suppose you have an excel file "WorkHours.csv" and you wanna use this excel document to prove to your manager you're being underpaid for all the billable time you're putting in.

```python
# Python Code
import pandas as pd

# Load a CSV file
csv_df = pd.read_csv("path/to/your_file.csv")

# Load an Excel file (first sheet by default)
excel_df = pd.read_excel("path/to/your_file.xlsx")

# Load a public Google Sheet
sheet_url = "https://docs.google.com/spreadsheets/d/your_sheet_id_here/export?format=csv"
gsheet_df = pd.read_csv(sheet_url)

# Display heads to verify
print("CSV Data:\n", csv_df.head(), "\n")
print("Excel Data:\n", excel_df.head(), "\n")
print("Google Sheet Data:\n", gsheet_df.head())

```

### Query from a Database

```python
# Python Code

import sqlite3
import pandas as pd

# Step 2: Connect to the database
conn = sqlite3.connect("sample_users.db")

# Step 3: Query the database
df = pd.read_sql("SELECT * FROM users", conn)
df.head()
```

### Google Cloud
You could use any cloud service to get data but the one I'm personally most familiar with is google's cloud services and particularly bigquery. This is virtually the same as querying from a database with the key difference being that the database is hosted on a cloud server instead of locally.

```python
#Python Code

from google.colab import auth
from google.cloud import bigquery
import pandas as pd

# Authenticate user (works in Colab)
auth.authenticate_user()

# Initialize client
client = bigquery.Client(project="bigquery-public-data")

# Query a public dataset
query = """
    SELECT name, SUM(number) as total
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    GROUP BY name
    ORDER BY total DESC
    LIMIT 5
"""

# Run query and convert to DataFrame
df = client.query(query).to_dataframe()
df

```

### Make Your Own (Bonus Tip)

If you're really pressed for finding data because you're manager is hounding you for data, you may be able to get away with simulating it, Monte Carlo Style. If you've ever heard the phrase monte carlo simulation, don't be scared. Monte Carlo simulations can be thought of performing a simple process with random variance. Think of a trend that's directionally accurate, add some noise or jitter to add some extra spice. Do some stress testing on said data. And hand that into your manager with the same enthusiasm VC startups have when they say they can corner 1% of a 5 trillion dollar market in 18 months with 20 engineers and 3 studio apartments.

## Conclusion

Your milage may vary in being able to make your boss happy. But, at the very least you have a place to start so you can get some preliminary analysis done. You still know how to analyze the data, right? Maybe 89th percentile is right where you belong.
