# tutorial-fraud-detection
Tutorial for Realtime Fraud Detection using Graphs, Query Workers & Stream Workers.


## Setup

| **Federation** | **Email** | **Passsword** |
|------------|----------|--------------|
| [Global Data Network](https://paas.gdn.macrometa.io/) | ebay@macrometa.io | `xxxxxxxx`| 


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

### Transactions (Edges)

```
LET e = [
	{"_from": "customers/Paul", "_to": "merchants/Just_Brew_It", "amount": 986, "status": "Undisputed", "time":"4/17/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Starbucks", "amount": 239, "status": "Undisputed", "time":"5/15/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Sears", "amount": 475, "status": "Undisputed", "time":"3/28/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Walmart", "amount": 654, "status": "Undisputed", "time":"3/20/2014"},
	{"_from": "customers/Jean", "_to": "merchants/Soccer_for_the_City", "amount": 196, "status": "Undisputed", "time":"7/24/2014"},
	{"_from": "customers/Jean", "_to": "merchants/Abercrombie", "amount": 502, "status": "Undisputed", "time":"4/9/2014"},
	{"_from": "customers/Jean", "_to": "merchants/Walmart", "amount": 848, "status": "Undisputed", "time":"5/29/2014"},
	{"_from": "customers/Jean", "_to": "merchants/Amazon", "amount": 802, "status": "Undisputed", "time":"3/11/2014"},
	{"_from": "customers/Jean", "_to": "merchants/Subway", "amount": 203, "status": "Undisputed", "time":"3/27/2014"},
	{"_from": "customers/Dan", "_to": "merchants/McDonalds", "amount": 35, "status": "Undisputed", "time":"1/23/2014"},
	{"_from": "customers/Dan", "_to": "merchants/McDonalds", "amount": 605, "status": "Undisputed", "time":"1/27/2014"},
	{"_from": "customers/Dan", "_to": "merchants/Soccer_for_the_City", "amount": 62, "status": "Undisputed", "time":"9/17/2014"},
	{"_from": "customers/Dan", "_to": "merchants/Amazon", "amount": 141, "status": "Undisputed", "time":"11/14/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Amazon", "amount": 134, "status": "Undisputed", "time":"4/14/2014"},
	{"_from": "customers/Marc", "_to": "merchants/American_Apparel", "amount": 336, "status": "Undisputed", "time":"4/3/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Walmart", "amount": 964, "status": "Undisputed", "time":"3/22/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Sears", "amount": 430, "status": "Undisputed", "time":"8/10/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Soccer_for_the_City", "amount": 11, "status": "Undisputed", "time":"9/4/2014"},
	{"_from": "customers/John", "_to": "merchants/Soccer_for_the_City", "amount": 545, "status": "Undisputed", "time":"10/6/2014"},
	{"_from": "customers/John", "_to": "merchants/Sprint", "amount": 457, "status": "Undisputed", "time":"10/15/2014"},
	{"_from": "customers/John", "_to": "merchants/Justice", "amount": 468, "status": "Undisputed", "time":"7/29/2014"},
	{"_from": "customers/John", "_to": "merchants/American_Apparel", "amount": 768, "status": "Undisputed", "time":"11/28/2014"},
	{"_from": "customers/John", "_to": "merchants/Just_Brew_It", "amount": 921, "status": "Undisputed", "time":"3/12/2014"},
	{"_from": "customers/Zoey", "_to": "merchants/McDonalds", "amount": 740, "status": "Undisputed", "time":"12/15/2014"},
	{"_from": "customers/Zoey", "_to": "merchants/Abercrombie", "amount": 510, "status": "Undisputed", "time":"11/27/2014"},
	{"_from": "customers/Zoey", "_to": "merchants/Just_Brew_It", "amount": 414, "status": "Undisputed", "time":"1/20/2014"},
	{"_from": "customers/Zoey", "_to": "merchants/Amazon", "amount": 721, "status": "Undisputed", "time":"7/17/2014"},
	{"_from": "customers/Zoey", "_to": "merchants/Subway", "amount": 353, "status": "Undisputed", "time":"10/25/2014"},
	{"_from": "customers/Ava", "_to": "merchants/Sears", "amount": 681, "status": "Undisputed", "time":"12/28/2014"},
	{"_from": "customers/Ava", "_to": "merchants/Walmart", "amount": 87, "status": "Undisputed", "time":"2/19/2014"},
	{"_from": "customers/Ava", "_to": "merchants/American_Apparel", "amount": 533, "status": "Undisputed", "time":"8/6/2014"},
	{"_from": "customers/Ava", "_to": "merchants/American_Apparel", "amount": 723, "status": "Undisputed", "time":"1/8/2014"},
	{"_from": "customers/Ava", "_to": "merchants/Just_Brew_It", "amount": 627, "status": "Undisputed", "time":"5/20/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Soccer_for_the_City", "amount": 75, "status": "Undisputed", "time":"9/4/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Walmart", "amount": 231, "status": "Undisputed", "time":"7/12/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Soccer_for_the_City", "amount": 924, "status": "Undisputed", "time":"10/4/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Just_Brew_It", "amount": 742, "status": "Undisputed", "time":"8/12/2014"},
	{"_from": "customers/Mia", "_to": "merchants/Soccer_for_the_City", "amount": 276, "status": "Undisputed", "time":"12/24/2014"},
	{"_from": "customers/Mia", "_to": "merchants/Starbucks", "amount": 66, "status": "Undisputed", "time":"4/16/2014"},
	{"_from": "customers/Mia", "_to": "merchants/McDonalds", "amount": 467, "status": "Undisputed", "time":"12/23/2014"},
	{"_from": "customers/Mia", "_to": "merchants/Sears", "amount": 830, "status": "Undisputed", "time":"3/13/2014"},
	{"_from": "customers/Mia", "_to": "merchants/Amazon", "amount": 240, "status": "Undisputed", "time":"7/09/2014"},
	{"_from": "customers/Mia", "_to": "merchants/Soccer_for_the_City", "amount": 164, "status": "Undisputed", "time":"12/26/2014"},
	{"_from": "customers/Madison", "_to": "merchants/McDonalds", "amount": 630, "status": "Undisputed", "time":"10/6/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Abercrombie", "amount": 19, "status": "Undisputed", "time":"7/29/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Subway", "amount": 352, "status": "Undisputed", "time":"12/16/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Amazon", "amount": 147, "status": "Undisputed", "time":"8/3/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Walmart", "amount": 91, "status": "Undisputed", "time":"6/29/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Apple_Store", "amount": 1021, "status": "Disputed", "time":"7/18/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Urban_Outfitters", "amount": 1732, "status": "Disputed", "time":"5/10/2014"},
	{"_from": "customers/Paul", "_to": "merchants/RadioShack", "amount": 1415, "status": "Disputed", "time":"4/1/2014"},
	{"_from": "customers/Paul", "_to": "merchants/Macys", "amount": 1849, "status": "Disputed", "time":"12/20/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Apple_Store", "amount": 1914, "status": "Disputed", "time":"7/18/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Urban_Outfitters", "amount": 1424, "status": "Disputed", "time":"5/10/2014"},
	{"_from": "customers/Marc", "_to": "merchants/RadioShack", "amount": 1721, "status": "Disputed", "time":"4/1/2014"},
	{"_from": "customers/Marc", "_to": "merchants/Macys", "amount": 1003, "status": "Disputed", "time":"12/20/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Apple_Store", "amount": 1149, "status": "Disputed", "time":"7/18/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Urban_Outfitters", "amount": 1152, "status": "Disputed", "time":"5/10/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/RadioShack", "amount": 1984, "status": "Disputed", "time":"4/1/2014"},
	{"_from": "customers/Olivia", "_to": "merchants/Macys", "amount": 1790, "status": "Disputed", "time":"12/20/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Apple_Store", "amount": 1923, "status": "Disputed", "time":"7/18/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Urban_Outfitters", "amount": 1375, "status": "Disputed", "time":"5/10/2014"},
	{"_from": "customers/Madison", "_to": "merchants/RadioShack", "amount": 1369, "status": "Disputed", "time":"4/1/2014"},
	{"_from": "customers/Madison", "_to": "merchants/Macys", "amount": 1816, "status": "Disputed", "time":"12/20/2014"}
]
FOR txn IN e
	INSERT txn IN txns
  
```

## Stream Workers

### txn-generator

This stream app send 5 transactions per second continually and repeats. Sample transactions are available in `dataset` section.

#### Query `sale_remove_txns`
```
FOR txn IN txns LIMIT 5
REMOVE txn IN txns RETURN OLD
```

#### Worker Code:
```
@App:name("txn-generator")
@App:description("This worker generates transactions on the base of data set")

define trigger TxnTrigger at every 1 seconds;

@sink(type='restql-call', restql.name="select_remove_txns", sink.id="txn-gen", ignore.params="true")
define stream restqlStream(value long);

-- json or passthrough
@source(type='restql-call-response', sink.id="txn-gen", @map(type="json"))
define stream restqlStreamResponse(_from string, _to string, amount int, status string, time string);

@sink(type='c8streams', stream="txns_stream", replication.type="global")
define stream txns_stream(_from string, _to string, amount int, status string, time string);

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

@sink(type='restql-call',restql.name="fraud_detection",sink.id="txn-fraud", ignore.params = "true")
define stream restqlStream(value long);

-- json or passthrough
@source(type='restql-call-response',sink.id="txn-fraud", @map(type="json"))
define stream restqlStreamResponse(merchant string);

@store(type='c8db', collection="culpable_merchants", replication.type="global", @map(type='json'))
define table culpable_merchants(merchant string, time string);

select eventTimestamp() as value
  from txns_disputed
insert into restqlStream;

select merchant, time:currentTime() as time
  from restqlStreamResponse
insert into culpable_merchants;
```


## Query Worker

Zero in on `culpable merchant`.

**fraud_detection**:

```js
	// Query to identify culpable merchant
	
	LET suspects = FLATTEN(
	    FOR t IN txns
		FILTER t.status == "Disputed"
		FOR prev IN txns
		    FILTER prev._from == t._from AND prev.time < t.time AND prev.status == "Undisputed"
		    COLLECT customer = t._from INTO info
		    RETURN (FOR merchant IN info[*].prev._to RETURN DISTINCT merchant)  
	)
	FOR suspect IN suspects
	    COLLECT merchant = suspect WITH COUNT INTO mentions
	    SORT mentions DESC
	    //RETURN MERGE(DOCUMENT(merchant), {"mentions": mentions})
	    LIMIT 1
	    RETURN DOCUMENT(merchant)
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
