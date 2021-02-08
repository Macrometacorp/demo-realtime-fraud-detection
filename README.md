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

### customers-generator
```
@App:name("customers-generator")
@App:description('Generates customers info.')

define trigger genCustomerTrigger at every 1 sec;

@store(type='c8db', collection='customers', replication.type="global", @map(type='json'))
define table customers(_key string, age long, gender string, name string);

select pii:fake('Name', "NAME_FIRSTNAME", true) as _key,
       math:round(math:rand()*70 + 15) as age,
       ifThenElse(math:round(math:rand()*10)%2 == 0, 'male', 'female') as gender,
       pii:fake('Name', "NAME_FIRSTNAME", false) as name
  from genCustomerTrigger
insert into customers;
```

### merchants-generator
```
@App:name("merchants-generator")
@App:description('Generates merchants info.')

define trigger genMerchantsTrigger at every 1 sec;

@store(type='c8db', collection='merchants', replication.type="global", @map(type='json'))
define table merchants(_key string, name string, address string, street string);

select pii:fake('CompanyName', "COMPANY_NAME", true) as _key,
       pii:fake('CompanyName', "COMPANY_NAME") as name,
       str:concat(pii:fake('City', "ADDRESS_CITY"), ", ", pii:fake('Zipcode', "ADDRESS_ZIPCODE")) as address,
       pii:fake('Street', "ADDRESS_STREETADDRESS") as street
  from genMerchantsTrigger
insert into merchants;
```

### txn-generator

This stream app send 1 transactions per second continually and repeats. 

```
@App:name("txn-generator")
@App:description("This worker generates transactions on the base of data set")

define trigger TxnTrigger at every 1 seconds;

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

```
@App:name("fraud_detection")
@App:description("This stream worker call fraud_detection query worker (aka restql) to detect fraud")

@source(type='c8db', collection="txns_disputed", replication.type="global", @map(type='json'))
define stream txns_disputed(_from string, _to string, amount int, status string, time string);

@sink(type='restql-call',restql.name="fraud_detection",sink.id="txn-fraud", ignore.params = "false")
define stream restqlStream(time string);

-- json or passthrough
@source(type='restql-call-response',sink.id="txn-fraud", stream="restqlStreamResponse", @map(type="json"))
define stream restqlStreamResponse(merchant object);

@store(type='c8db', collection="culpable_merchants", replication.type="global", @map(type='json'))
define table culpable_merchants(merchant object, time string);

select time
  from txns_disputed
insert into restqlStream;

select merchant, time:currentTimestamp() as time
  from restqlStreamResponse
insert into culpable_merchants;
```

## Query for random transactional data

**random_data_generator**:

```js
FOR i IN 1..50
  LET cus = (FOR cus IN customers SORT RAND()*10/10 LIMIT 1 RETURN cus._key)
  LET mer = (FOR mer IN merchants SORT RAND()*10/10 LIMIT 1 RETURN mer._key)
  RETURN {_from : CONCAT_SEPARATOR("/", "customers", FIRST(cus)), 
            _to : CONCAT_SEPARATOR("/", "merchants", FIRST(mer)), 
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

	LET end = @time
	LET start = DATE_ISO8601(DATE_TIMESTAMP(@time) - 3 * 24 * 60 * 60 * 1000) // end - 3 days

	// For each disputed transaction in the time period, find the time of the earliest for each customer.
	LET customers = (
	    FOR t IN txns_disputed
		FILTER start < t.time AND t.time <= end
		COLLECT customer = t._from
		AGGREGATE time = MIN(t.time)
		SORT (customer == CONCAT("customers/", @customer)) DESC
		RETURN [customer, time]
	)

	// Determine the suspect merchants based on each customer's transactions.
	LET suspects = (
	    FOR entry IN customers
		LET customer = entry[0]
		LET time = entry[1]
		RETURN (
		    FOR prev IN txns_undisputed
			FILTER prev._from == customer AND prev.time < time
			RETURN DISTINCT prev._to
		)
	)
	LET first = suspects[0] // suspects for the complaining customer

	// Find a merchant suspected for the most customers.
	FOR suspect IN FLATTEN(suspects)
	    FILTER suspect IN first
	    COLLECT merchant = suspect WITH COUNT INTO mentions
	    SORT mentions DESC
	    // RETURN MERGE(DOCUMENT(merchant), {"mentions": mentions})
	    LIMIT 1
	    RETURN {"merchant": merchant}
```

## Collection Indexes

### Collection: culpable_merchants

```
ID	Type	Unique	Sparse	Deduplicate	Selectivity Est.	Fields	Name	Action
0	primary	true	false	n/a	100.00%	_key	primary	
119657888	ttl	false	true	n/a	96.30%	time	ttl_time_index	
```

### Collection: txns_disputed

```
ID	Type	Unique	Sparse	Deduplicate	Selectivity Est.	Fields	Name	Action
0	primary	true	false	n/a	100.00%	_key	primary	
2	edge	false	false	n/a	1.30%	_from, _to	edge	
121060565	ttl	false	true	n/a	33.59%	time	ttl-idx-259200	
138242033	persistent	false	false	false	33.71%	time	time	
```

### Collection: txns_undisputed

```
ID	Type	Unique	Sparse	Deduplicate	Selectivity Est.	Fields	Name	Action
0	primary	true	false	n/a	100.00%	_key	primary	
2	edge	false	false	n/a	0.34%	_from, _to	edge	
121058843	ttl	false	true	n/a	12.47%	time	ttl-idx-259200	
138410562	persistent	false	false	false	49.93%	_from, time	suspects
```
