# TRANSACTIONS-GRAPH-GENERATOR
`Before starting, please manually create folders data, output and log in the root of the repo`

[Medium](https://medium.com/@mgrin/how-to-generate-a-huge-financial-graph-with-money-laundering-patterns-5c3e490dd683) article.

A big graph generator for transactions graph. As output you'll get:
* a csv with transactions
* 3 csv, one for clients, one for companies, one for ATMs, containing information about nodes of the graph

Also generates some patterns inside the graph (FLow, Circle, Time patterns)

Theoretically supports generation of any sized graph (kind of optimized, but not tested n graph more than 100000 nodes and > 10^9 transactions)

# Resulting files structure
The generator outputs its result to the data folder (can be changed using parameters). These are different files with data from different steps.
To merge these files into 4 well defined csv, you have to run the transformation script from `scripts` folder:
```
./scripts/transform.sh
```
This will put 4 csv files concatenated by type, with randomized transactions (not to have them one after another, for example for patterns all transactions of one pattern follow each other before that script and are randomized after).

# How to use
## Installation
* You'll need `pipenv` installed
* `pipenv install`
* `mkdir -p data output logs`

## Config-less
```
pipenv run python generateGraph 100

./sctipts/transform.sh
```
Will generate a graph with 100 clients, 1 ATM and 2 companies.  Number of transactions is following a given distribution (look code to know more)

## Config-full
All configurations are described in `generateGraph.py` file
```
pipenv run python --data=./myOwnFolder --probs=0.01,0.001,0.03,0.005 --steps=nodes,edges,transactions,patterns --batch-size=5000 10000

./scripts/transform.sh
```
* `--data` : folder to store generated data
* `--probs` : list of connection creation probabilities. Format: client-client,client-company,client-atm,company-client
* `--steps` : Steps to do. possible values (comma - separated): nodes, edges, transactions, patterns. Should be ordered (transaction swill not be generated before edges, for example)
* `--batch-size` : While generating, data is written to disk by batches of given size. An element in a batch is a line in CSV file. Also, batch size controls frequency of logs. More batch size is more memory you need (will be used to store generated data) but should work faster (in theory, not in practice :))

# Data and Patterns
## Client
* id
* first_name
* last_name
* age
* email
* occupation
* political_views
* nationality
* university
* academic_degree
* address
* postal_code
* country
* city

## Company
* id
* type
* name
* country

## ATM
* id
* latitude
* longitude

## Transaction
* id
* source (points by ID to other node types)
* target (points by ID to other node types)
* date
* time
* amount
* currency

## Patterns
There are 3 types of pattern generated:
### Flow
Money starts from node A, goes through K levels, with K_N nodes on each level, and comes to a node B without a small sum payed to all network participants for their "work". Parameners:
* K (number of layers): randint(2, 6)
* K_N (number of nodes on layer K): randint(1, 8)
* Total payback (payed to intermediate nodes): 0.1 * random() * totalSum

So `TOTAL_SUM` exits from NodeA and `TOTAL_SUM * (1 - 0.1 * random())` comes to NodeB.

All transactions between layers are delayed by a random time (not that random, like between couple of seconds and couple of days)
### Circular
Money starts from node A, goes through N nodes one by one and comes back to node A without a small sum payed to all network participants for their "work". Parameters:
* N (number of nodes in the circle): randint(1, 8)
* Total payback (payed to intermediate nodes): 0.1 * random() * totalSum

Again, transactions a delayed

### Time
Exactly same amount goes from node A to node B multiple times separated by T equal time intervals. Parameters:
* T (number of time intervals): randint(5, 50)
