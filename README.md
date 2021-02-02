# tutorial-fraud-detection
Tutorial for Realtime Fraud Detection using Graphs, Query Workers & Stream Workers.


## Setup

| **Federation** | **Email** | **Passsword** |
|------------|----------|--------------|
| [Global Data Network](https://gdn.paas.macrometa.io/) | ebay@macrometa.io | `xxxxxxxx`| 


## Sample Dataset

### Customers

```
let cust = [
    {"_key":"Marc","age":30,"gender":"man","name":"Marc"},
    {"_key":"Zoey","age":52,"gender":"woman","name":"Zoey"},
    {"_key":"Dan","age":23,"gender":"man","name":"Dan"},
    {"_key":"Jean","age":48,"gender":"man","name":"Jean"},
    {"_key":"Ava","age":23,"gender":"woman","name":"Ava"},
    {"_key":"Madison","age":37,"gender":"woman","name":"Madison"},
    {"_key":"Mia","age":51,"gender":"woman","name":"Mia"},
    {"_key":"Paul","age":50,"gender":"man","name":"Paul"},
    {"_key":"Olivia","age":58,"gender":"woman","name":"Olivia"},
    {"_key":"John","age":31,"gender":"man","name":"John"}
]

for customer in cust 
    insert customer in customers

```
### Merchants

```
LET m = [
	{"_key": "Amazon", "name": "Amazon", "street": "2626 Wilkinson Court", "address":"San Bernardino, CA 92410"},
	{"_key": "Abercrombie", "name": "Abercrombie", "street": "4355 Walnut Street", "address":"San Bernardino, CA 92410"},
	{"_key": "Walmart", "name": "Walmart", "street": "2092 Larry Street", "address":"San Bernardino, CA 92410"},
	{"_key": "McDonalds", "name": "McDonalds", "street": "1870 Caynor Circle", "address":"San Bernardino, CA 92410"},
	{"_key": "American_Apparel", "name": "American_Apparel", "street": "1381 Spruce Drive", "address":"San Bernardino, CA 92410"},
	{"_key": "Just_Brew_It", "name": "Just_Brew_It", "street": "826 Anmoore Road", "address":"San Bernardino, CA 92410"},
	{"_key": "Justice", "name": "Justice", "street": "1925 Spring Street", "address":"San Bernardino, CA 92410"},
	{"_key": "Sears", "name": "Sears", "street": "4209 Elsie Drive", "address":"San Bernardino, CA 92410"},
	{"_key": "Soccer_for_the_City", "name": "Soccer for the City", "street": "86 D Street", "address":"San Bernardino, CA 92410"},
	{"_key": "Sprint", "name": "Sprint", "street": "945 Kinney Street", "address":"San Bernardino, CA 92410"},
	{"_key": "Starbucks", "name": "Starbucks", "street": "3810 Apple Lane", "address":"San Bernardino, CA 92410"},
	{"_key": "Subway", "name": "Subway", "street": "3778 Tenmile Road", "address":"San Bernardino, CA 92410"},
	{"_key": "Apple_Store", "name": "Apple Store", "street": "349 Bel Meadow Drive", "address":"Kansas City, MO 64105"},
	{"_key": "Urban_Outfitters", "name": "Urban Outfitters", "street": "99 Strother Street", "address":"Kansas City, MO 64105"},
	{"_key": "RadioShack", "name": "RadioShack", "street": "3306 Douglas Dairy Road", "address":"Kansas City, MO 64105"},
	{"_key": "Macys", "name": "Macys", "street": "2912 Nutter Street", "address":"Kansas City, MO 64105"}
]
FOR merchant IN m
	insert merchant in merchants
  
```


## Stream Workers

### txn-generator

This stream app send 1 transactions per second continually and repeats. 

#### Worker Code:
```
@App:name("txn-generator")
@App:description("This worker generates transactions on the base of data set")

define trigger TxnTrigger at every 1 seconds;

--@sink(type='restql-call', restql.name="sale_remove_txns", sink.id="txn-gen", ignore.params="true")
@sink(type='restql-call', restql.name="random_data_generator", sink.id="txn-gen", ignore.params="true")
define stream restqlStream(value long);

-- json or passthrough
@source(type='restql-call-response', sink.id="txn-gen", @map(type="json"))
define stream restqlStreamResponse(_from string, _to string, amount int, status string, time string);

@sink(type='c8streams', stream="txns_stream", replication.type="global")
define stream txns_stream(_from string, _to string, amount int, status string, time string);

@store(type='c8db', collection="txns", replication.type="global")
define table txns(_from string, _to string, amount int, status string, time string);

select  eventTimestamp() as value
  from TxnTrigger
insert into restqlStream;

select _from, _to, amount, status, time
  from restqlStreamResponse
insert into txns_stream;
```


### txn-processor
This stream worker processes incoming transactions and populates two edge collections (`txns_disputed` and `txns_undisputed`) based on the transaction type.

#### Worker Code:
```
@App:name("txn-processor")
@App:description("This stream worker processes incoming transactions and populates two edge collections (txns_disputed and txns_undisputed)")

@source(type='c8streams', stream.list="txns_stream", replication.type="global", @map(type='json'))
define stream txns_stream(_from string, _to string, amount int, status string, time string);

@store(type='c8db', collection="txns_disputed", replication.type="global", @map(type='json'))
define table txns_disputed(_from string, _to string, amount int, status string, time string);

@store(type='c8db', collection="txns_undisputed", replication.type="global", @map(type='json'))
define table txns_undisputed(_from string, _to string, amount int, status string, time string);

select _from, _to, amount, status, time 
  from txns_stream[status == "Disputed"]
insert into txns_disputed;

select _from, _to, amount, status, time
  from txns_stream[status == "Undisputed"]
insert into txns_undisputed;
```

### fraud-detector
This stream worker call `fraud_detection` query worker (aka restql) to detect fraud and write results to `culpable-merchants` collection. 
document format: 
```
{ 
  "date": "YYYY-MM-DD HH:mm:SS
  "merchant": "xxxxx"
}
```

#### Worker Code:
```
@App:name("fraud_detection")
@App:description("This stream worker call fraud_detection query worker (aka restql) to detect fraud")

@source(type='c8db', collection="txns_disputed", replication.type="global", @map(type='json'))
define stream txns_disputed(_from string, _to string, amount int, status string, time string);

@sink(type='restql-call',restql.name="fraud_detection",sink.id="txn-fraud", ignore.params = "false")
define stream restqlStream(time string, customer string);

-- json or passthrough
@source(type='restql-call-response',sink.id="txn-fraud", stream="restqlStreamResponse", @map(type="json"))
define stream restqlStreamResponse(merchant object);

@store(type='c8db', collection="culpable_merchants", replication.type="global", @map(type='json'))
define table culpable_merchants(merchant object, time string);

select time, _from as customer
  from txns_disputed
insert into restqlStream;

select merchant, time:currentTimestamp() as time
  from restqlStreamResponse
insert into culpable_merchants;
```

## Query for random transactional data

**random_data_generator**:

```js
FOR cus IN customers SORT RAND()*10/10 LIMIT 1
  FOR mer IN merchants SORT RAND()*10/10 LIMIT 1 
RETURN {_from : CONCAT_SEPARATOR("/", "customers", cus._key), 
          _to :  CONCAT_SEPARATOR("/", "merchants", mer._key), 
       amount : FLOOR((RAND() + 1) * 100), 
       status : (FLOOR(RAND()*10)%4) == 0 ? "Disputed" : "Undisputed",
         time : DATE_ISO8601(DATE_NOW())
}
```

## Query Worker

Zero in on `culpable merchant`.

**fraud_detection**:

```js
	// Query to identify culpable merchant
	
LET suspects = FLATTEN(
 FOR t IN txns_disputed
 FILTER t.time == @time
   FOR prev IN txns_undisputed
	    FILTER prev._from == t._from AND prev.time < t.time
	    COLLECT customer = t._from INTO info
	    RETURN (FOR merchant IN info[*].prev._to RETURN DISTINCT merchant)  
	)
 FOR suspect IN suspects
    COLLECT merchant = suspect WITH COUNT INTO mentions
    SORT mentions DESC
    LIMIT 1
    RETURN {merchant}
```

## Additional Notes

1. Identify `Fradulent Txns`

    ```js
    // Query to identify all fradulent transactions
    FOR txn IN txns
        FILTER txn.status=="Disputed"
        RETURN txn

    ```

2. Identify `Point of Origin`

    ```js

    // Query to identify fraud point of orgin
    FOR x IN txns FILTER x.status == "Disputed"
    FOR y in txns FILTER y.status == "Undisputed"  
            FILTER y.time < x.time AND y._to != x._to AND y._from == x._from
        FOR customer in customers FILTER customer._id == y._from
        FOR merchant in merchants FILTER merchant._id == y._to
        SORT customer.name
        RETURN  {
            customer: customer.name, 
            merchant: merchant.name, 
            amount: y.amount, 
            date: y.time}

    ```

3. Zero in on `culpable merchant` - Go through all current disputed transactions and find a common merchant where all these customers have made a transaction in the recent past. The idea is that this common merchant is where the credit card data of these customers of disputed transactions is stolen.

    ```js
        // Query to identify culpable merchant
	    FOR x IN txns FILTER x.status == "Disputed"
	    FOR y in txns FILTER y.status == "Undisputed"  
		    FILTER y.time < x.time AND y._to != x._to AND y._from == x._from
		FOR customer in customers FILTER customer._id == y._from
		FOR merchant in merchants FILTER merchant._id == y._to
		SORT customer.name
		RETURN DISTINCT y
	
    ```
