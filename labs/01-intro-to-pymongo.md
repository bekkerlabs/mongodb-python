# Intro to Pymongo

MongoDB with PyMongo Lab

## Installing

Before we continue, we should ensure that pymongo is installed:

```
$ pip install pymongo --user
```

And also make sure that we can reach mongodb:

```
$ nc -vz localhost 27017
```

### Making a Connection

Making a connection without authentication (not our case):

```
>>> from pymongo import MongoClient
>>> uri = 'mongodb://mongodb.domain.com:27017/'
>>> client = MongoClient(uri)
>>> client.list_database_names()
['admin', 'config', 'local']
```

Making a connection with authentication:

```
>>> from pymongo import MongoClient
>>> uri = 'mongodb://root:bekkerlabs@localhost:27017/'
>>> client = MongoClient(uri)
>>> client.list_database_names()
['admin', 'config', 'local']
```

We can also use environment variables, to test if we have the values in our environment:

```
>>> import os

>>> os.environ['MONGODB_HOST']
'localhost'
>>> os.environ['MONGODB_USER']
'root'
>>> os.environ['MONGODB_PASS']
'bekkerlabs'
>>> os.environ['MONGODB_PORT']
'27017'
```

Now we can use our environment variables:

```
>>> import os
>>> mdb_user = os.environ['MONGODB_USER']
>>> mdb_pass = os.environ['MONGODB_PASS']
>>> mdb_host = os.environ['MONGODB_HOST']
>>> mdb_port = os.environ['MONGODB_PORT']
>>> uri = 'mongodb://{user}:{passwd}@{host}:{port}/'.format(user=mdb_user, passwd=mdb_pass, host=mdb_host, port=mdb_port)
>>> client = MongoClient(uri)
>>> client.list_database_names()
['admin', 'config', 'local']
```

### Listing Databases


```
>>> client = MongoClient(uri)
>>> client.list_database_names()
['admin', 'config', 'local']
```

### Listing Collections

```
>>> client = MongoClient(uri)
>>> db = client.shared_db
>>> db.list_collection_names()
[]
```

### Write One Document 

Create a database `store_db`, use the `transactions` collection and write a document to it.

```
>>> db = client.store_db
>>> transactions = db.transactions
>>> doc_data = {
    'store_name': 'sportsmans', 
    'branch_name': 'tygervalley', 
    'account_id': 'sns_03821023', 
    'total_costs': 109.20, 
    'products_purchased': ['cricket bat', 'cricket ball', 'sports hat'], 
    'purchase_method': 
    'credit card'
}
>>> response = transactions.insert_one(doc_data)
>>> response.inserted_id
ObjectId('5cad16a5a5f3826f6f046d74')
```

We can verify that the collection is present:

```
>>> db.collection_names()
[u'transactions']
```

### Write Many Documents

We can batch up our writes:

```
>>> transaction_1 = {
    'store_name': 'sportsmans', 'branch_name': 'tygervalley', 
    'account_id': 'sns_09121024', 'total_costs': 129.84, 
    'products_purchased': ['sportsdrink', 'sunglasses', 'sports illustrated'], 
    'purchase_method': 'credit card'
}
>>> transaction_2 = {
    'store_name': 'burger king', 'branch_name': 
    'somerset west', 'account_id': 'bk_29151823', 
    'total_costs': 89.99, 'products_purchased': ['cheese burger', 'pepsi'], 
    'purchase_method': 'cash'
}
>>> transaction_3 = {
    'store_name': 'game', 'branch_name': 'bellvile', 'account_id': 'gm_49121229', 
    'total_costs': 499.99, 'products_purchased': ['ps4 remote'], 
    'purchase_method': 'cash'
}
>>> response = transactions.insert_many([transaction_1, transaction_2, transaction_3])
>>> response.inserted_ids
[ObjectId('5cad18d4a5f3826f6f046d75'), ObjectId('5cad18d4a5f3826f6f046d76'), ObjectId('5cad18d4a5f3826f6f046d77')]
```

### Find One Document:

```
>>> transactions.find_one({'account_id': 'gm_49121229'})
{u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}
```

### Find Many Documents:

```
>>> response = transactions.find({'purchase_method': 'cash'})
>>> [doc for doc in response]
[{u'account_id': u'bk_29151823', u'store_name': u'burger king', u'purchase_method': u'cash', u'branch_name': u'somerset west', u'products_purchased': [u'cheese burger', u'pepsi'], u'_id': ObjectId('5cad18d4a5f3826f6f046d76'), u'total_costs': 89.99}, {u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}]
```

Or filtering down the results to only the account id:

```
>>> response = transactions.find({'purchase_method': 'cash'})
>>> [doc['account_id'] for doc in response]
[u'bk_29151823', u'gm_49121229']
```

MongoDB also has a count method:

```
>>> transactions.find({'purchase_method': 'cash'}).count()
2
```

Query with the AND condition. SQL equivalent: `where branch_name = 'tygervalley' AND account_id = 'sns_03821023'`

```
>>> response = transactions.find({ '$and': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}]
```

Query with the OR condition. SQL equivalent: `where branch_name = 'tygervalley' OR account_id = 'sns_03821023'`:

```
>>> response = transactions.find({ '$or': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}, {u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cb188a3df585e003c976d5e'), u'total_costs': 129.84}]
```

Combining it:

```
>>> response = transactions.find({ 'total_costs': {'$gt': 120}, '$or': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cb188a3df585e003c976d5e'), u'total_costs': 129.84}]
```

Other condition operations include:

```
lt  - Less Than
lte - Less Than Equals
gt  - Greater Than
gte - Greater Than Equals
ne  - Not Equals
```

## Range Queries:

```
>>> import datetime
>>> new_posts = [{"author": "Mike","text": "Another post!","tags": ["bulk", "insert"],"date": datetime.datetime(2009, 11, 12, 11, 14)},{"author": "Eliot","title": "MongoDB is fun","text": "and pretty easy too!","date": datetime.datetime(2009, 11, 10, 10, 45)}]
>>> db.posts.insert(new_posts)
[ObjectId('5cb439c0f90e2e002a164d15'), ObjectId('5cb439c0f90e2e002a164d16')]

>>> for post in db.posts.find():
...     post
...
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('5cb439c0f90e2e002a164d15'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}
{u'date': datetime.datetime(2009, 11, 10, 10, 45), u'text': u'and pretty easy too!', u'_id': ObjectId('5cb439c0f90e2e002a164d16'), u'author': u'Eliot', u'title': u'MongoDB is fun'}

>>> d = datetime.datetime(2009, 11, 11, 12)
>>> for post in db.posts.find({"date": {"$lt": d}}).sort("author"):
...     post
...
{u'date': datetime.datetime(2009, 11, 10, 10, 45), u'text': u'and pretty easy too!', u'_id': ObjectId('5cb439c0f90e2e002a164d16'), u'author': u'Eliot', u'title': u'MongoDB is fun'}
```

## Updates

Let's say we want to change a transactions payment method from credit card to account:

```
>>> transactions.find_one({'account_id':'sns_03821023'})
{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}```
```

Update the purchase_method to account:

```
>>> transactions.update( {'account_id': 'sns_03821023'}, {'$set': {'purchase_method': 'account'}})
{'updatedExisting': True, u'nModified': 1, u'ok': 1.0, u'n': 1}
```

Verify:

```
>>> transactions.find_one({'account_id':'sns_03821023'})
{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'account', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}
```

This examples is only intended for a single document and mongodb only updates one document at a time. To update more than one at a time:

```
transactions.update( {'k': 'v'}, {'$set': {'k': 'new_v'}},{multi:true})
```


### Filters

Find all the documents with purchase price > 120:

```
>>> response = transactions.find({'total_costs': {'$gt': 120}})
>>> [doc for doc in response]
[{u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cad18d4a5f3826f6f046d75'), u'total_costs': 129.84}, {u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}]
```

### Projections

Select specific fields from the returned response:

```
>>> response = transactions.find({}, {'branch_name': 'tygervalley', 'purchase_method': 'credit card'})
>>> [doc for doc in response]
[{u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'_id': ObjectId('5cad16a5a5f3826f6f046d74')}, {u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'_id': ObjectId('5cad18d4a5f3826f6f046d75')}, {u'purchase_method': u'cash', u'branch_name': u'somerset west', u'_id': ObjectId('5cad18d4a5f3826f6f046d76')}, {u'purchase_method': u'cash', u'branch_name': u'bellvile', u'_id': ObjectId('5cad18d4a5f3826f6f046d77')}]
```

### Sorting Documents

Sorting Documents in Descending Order:

```
>>> from pymongo import MongoClient, DESCENDING
>>> response = transactions.find().sort('total_costs', DESCENDING)
>>> ['Products: {}, Price: {}'.format(doc['products_purchased'], doc['total_costs']) for doc in response]
["Products: [u'ps4 remote'], Price: 499.99", "Products: [u'sportsdrink', u'sunglasses', u'sports illustrated'], Price: 129.84", "Products: [u'cricket bat', u'cricket ball', u'sports hat'], Price: 109.2", "Products: [u'cheese burger', u'pepsi'], Price: 89.99"]
```

### Aggregations

```
>>> agr = [{'$group': {'_id': 1, 'all': { '$sum': '$total_costs' }}}]
>> [a for a in transactions.aggregate(agr)]
[{u'all': 829.02, u'_id': 1}]
```

or:

```
>>> agr = [{'$group': {'_id': 1, 'all': { '$sum': '$total_costs' }}}]
>>> val = list(transactions.aggregate(agr))
>>> val
[{u'all': 829.02, u'_id': 1}]
```

Select fields to aggregate, eg. aggregate the costs for selected stores:

```
>>> agr = [{ '$match': {'$or': [ { 'store_name': 'sportsmans' }, { 'store_name': 'game' }] }}, { '$group': {'_id': 1, 'sum2stores': { '$sum': '$total_costs' } }}]
>>> [a for a in transactions.aggregate(agr)]
[{u'_id': 1, u'sum2stores': 739.03}]
```

### Limit Data Output

```
>>> response = transactions.find()
>>> [a['account_id'] for a in response]
[u'sns_03821023', u'sns_09121024', u'bk_29151823', u'gm_49121229']

>>> response = transactions.find().limit(2)
>>> [a['account_id'] for a in response]
[u'sns_03821023', u'sns_09121024']

>>> response = transactions.find().skip(1).limit(3)
>>> [a['account_id'] for a in response]
[u'sns_09121024', u'bk_29151823', u'gm_49121229']

>>> response = transactions.find().skip(1).limit(2)
>>> [a['account_id'] for a in response]
[u'sns_09121024', u'bk_29151823']

>>> response = transactions.find().skip(3).limit(1)
>>> [a['account_id'] for a in response]
[u'gm_49121229']
```

## Indexes

Create index:

```
>>> from pymongo import TEXT
>>> transactions.create_index([("store_name", TEXT)], name='store_index', default_language='english')
'store_index'
```

## Delete Documents:

Delete selected documents:

```
>>> transactions.remove({'account_id':'sns_03821029'})
{u'ok': 1.0, u'n': 2}
```

Delete all documents:

```
>>> transactions.remove()
```

### Drop Collections

```
>>> transactions.drop()
>>> db.collection_names()
[]
```
