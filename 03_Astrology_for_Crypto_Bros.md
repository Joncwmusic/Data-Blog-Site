# Astrology for Crypto Bros Dashboard Building (Part 1)

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
OR I could use my iteratively developed brain and find an automated solution. 
In fact, there are several ways to get that data without leaving your chair and that would be through various APIs.
In this project, I used coingecko as it was free. Feel free to peruse their documentation here if you’re crazy enough to try gambling on meme coins 
https://docs.coingecko.com/

