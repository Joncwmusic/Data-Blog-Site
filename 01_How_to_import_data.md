# How to import data sources with python

So you've survived another round of layoffs at the big tech company you've been cruising at the bare minimum for a while now. 
Instead of improving your skils or working on your fundamentals, you decided to watch videos about improving your skills trying to muster the motivation to change. 
You're doing great. Except, you're KPIs put you in the 89th percentile of staff and the bottom 10% were just given opportunities that aren't within the company. Ouch.
Now you have to get yourself back to at least the middle of the pack so upper management doesn't catch on to how much you've been slacking.
Your first task is to just get some preliminary data. Your boss doesn't know anything about the data on your internal servers, cloud storage, or even the data in the csvs he has saved locally.
He thinks API means Asian/Pacific Islander because he spent 5 years in HR before becoming a tech hiring manager and eventually, your direct report.


## Learning Goals

The goal is simply to lay out the basic options you have to gather data so you can analyse and predict to your hearts content so you're boss can give you high marks on your performance review.
You'll learn how to:

- Import data from public datasets online
- Scrape data from the web with beautiful soup
- Get data via an API
- Use pandas to read in csv files
- Source data directly from a SQLite Database
- Get data from a cloud based data source

With enough time, energy, effort, and caffiene almost anything is possible. Now let's get into it.

### Public Datasets

Public Datasets are easy to work with. You just need a direct url to the dataset and there are several sites whose sole purpose is to give people data. That's great for you, the newly inspired overachiever. It gives you a way to get your reps in and all you have to do is copy paste like the majority of your career has been and still will be, even as you copy paste from the LLM you used for your last report.

Data.gov is great to get you started

```
# Python Code:
import pandas as pd

dataset_url_gov = 'https://data.ny.gov/api/views/e2u6-bmnn/rows.csv'

gov_df = pd.read_csv(dataset_url_gov)
gov_df.head()
```
And now you have data that you can analyse to your hearts content.

### Web Scraping

Web scraping is a little harder to get clean data from but it's better than nothing if you're in a pinch. You can take tables directly from sites, or grab the same piece of info from different webpages. Sometimes you'll even find a juicy json file with a ton of metadata. Really, it depends on the site and your patience.

Let's go through a couple basic examples.

Maybe a table from Wikipedia.

### API
### Spreadsheets
### Query from a Database
### Google Cloud
### Make Your Own (Bonus Tip) 
