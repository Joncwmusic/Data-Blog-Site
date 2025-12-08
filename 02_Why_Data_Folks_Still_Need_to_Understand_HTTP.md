# Why Data Scientists Still Need to Know About HTTP

![NYEDB Image](NYRDB_Banner.png)

Data scientists, data analysts, data engineers, and... the business analysts, still stuck in their spreadsheets *ick*, come from all kinds of backgrounds. 
Some data scientists and engineers are former software engineers that realize they're better suited to moving numbers around than designing a functional UI. 
Meanwhile others had a better time studying mathematics and gagged everytime they picked up a TI-83
while they hand wrote their proofs before begrudgingly transfering their writings into `LaTeX`.
Now you might know some basic SQL and python and might've vibe coded your way through the last 2 years with scripts that your real software engineer colleagues have openly laughed at.
But now you're trying to skill up and that might mean learning a thing or two about how some of the internet actually works. After all, your servers are cloud based,
your data pipelines are based on a deprecated API endpoint barely holding on, and every time you use chatgpt you're still amazed your computer is doing that. 
(spoiler: *your* computer is not the one giving you an answer when you ask chatgpt: "How make data go brrrr.")

So let's talk about HTTP (and it's better sibling, HTTPS)

## What is HTTP?
HTTP stands for HyperText Transfer Protocol. It's a system for transfering data between a server and a client.
When I go to a website I'm not actually "going" to the website. Instead the website is giving my computer files containing the content I want to see.

It kind of looks like this:

- Bad python script in notebook sends request to server 

- Server says "yes you can have files per your request" and sends a response

- Response goes into your dataframe per the code in your python notebook

Any request usually has these componenets:

- The method: namely `GET` to retreieve data from the server, `POST` to upload data to the server, `PUT` to replace or create data in the server, and `DELETE` to remove data from the server
- The URL: the resource to be retrieved or edited (this may include paramters for yielding the proper response)
- Headers: for access controls. Effectively it's your clearance for access to the server to begin with
- The body: for sending parameters (this may include the headers)

## So Why do YOU need to know it?

As companies continue to consolidate data teams, your capacity to wear many hats is essential.

## HTTP Methods

## Common Error Codes

## Bonus Tip
