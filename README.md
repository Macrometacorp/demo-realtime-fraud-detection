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

### Transactions (Edges)

```
LET transactionsData =[
  {
    "_from": "customers/Paul",
    "_to": "merchants/Just_Brew_It",
    "amount": 295,
    "status": "Undisputed",
    "time": "02/01/2021 23:02:15"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Starbucks",
    "amount": 288,
    "status": "Undisputed",
    "time": "10/08/2020 04:07:20"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Sears",
    "amount": 706,
    "status": "Undisputed",
    "time": "10/15/2020 18:19:10"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Walmart",
    "amount": 731,
    "status": "Undisputed",
    "time": "12/21/2020 17:43:36"
  },
  {
    "_from": "customers/Jean",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 317,
    "status": "Undisputed",
    "time": "10/17/2020 19:18:39"
  },
  {
    "_from": "customers/Jean",
    "_to": "merchants/Abercrombie",
    "amount": 254,
    "status": "Undisputed",
    "time": "11/17/2020 22:39:11"
  },
  {
    "_from": "customers/Jean",
    "_to": "merchants/Walmart",
    "amount": 482,
    "status": "Undisputed",
    "time": "11/06/2020 03:15:00"
  },
  {
    "_from": "customers/Jean",
    "_to": "merchants/Amazon",
    "amount": 215,
    "status": "Undisputed",
    "time": "01/18/2021 15:44:34"
  },
  {
    "_from": "customers/Jean",
    "_to": "merchants/Subway",
    "amount": 447,
    "status": "Undisputed",
    "time": "01/11/2021 08:39:22"
  },
  {
    "_from": "customers/Dan",
    "_to": "merchants/McDonalds",
    "amount": 949,
    "status": "Undisputed",
    "time": "10/28/2020 10:46:41"
  },
  {
    "_from": "customers/Dan",
    "_to": "merchants/McDonalds",
    "amount": 899,
    "status": "Undisputed",
    "time": "10/23/2020 10:46:59"
  },
  {
    "_from": "customers/Dan",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 398,
    "status": "Undisputed",
    "time": "01/22/2021 17:15:52"
  },
  {
    "_from": "customers/Dan",
    "_to": "merchants/Amazon",
    "amount": 744,
    "status": "Undisputed",
    "time": "10/09/2020 13:44:09"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Amazon",
    "amount": 490,
    "status": "Undisputed",
    "time": "11/02/2020 22:14:13"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/American_Apparel",
    "amount": 775,
    "status": "Undisputed",
    "time": "10/09/2020 04:16:34"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Walmart",
    "amount": 221,
    "status": "Undisputed",
    "time": "12/27/2020 00:31:58"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Sears",
    "amount": 316,
    "status": "Undisputed",
    "time": "10/25/2020 23:58:35"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 299,
    "status": "Undisputed",
    "time": "01/16/2021 07:49:27"
  },
  {
    "_from": "customers/John",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 721,
    "status": "Undisputed",
    "time": "11/05/2020 11:38:21"
  },
  {
    "_from": "customers/John",
    "_to": "merchants/Sprint",
    "amount": 903,
    "status": "Undisputed",
    "time": "11/06/2020 15:45:04"
  },
  {
    "_from": "customers/John",
    "_to": "merchants/Justice",
    "amount": 842,
    "status": "Undisputed",
    "time": "11/18/2020 02:01:35"
  },
  {
    "_from": "customers/John",
    "_to": "merchants/American_Apparel",
    "amount": 73,
    "status": "Undisputed",
    "time": "01/19/2021 15:32:27"
  },
  {
    "_from": "customers/John",
    "_to": "merchants/Just_Brew_It",
    "amount": 714,
    "status": "Undisputed",
    "time": "12/22/2020 04:30:25"
  },
  {
    "_from": "customers/Zoey",
    "_to": "merchants/McDonalds",
    "amount": 382,
    "status": "Undisputed",
    "time": "01/23/2021 16:17:40"
  },
  {
    "_from": "customers/Zoey",
    "_to": "merchants/Abercrombie",
    "amount": 600,
    "status": "Undisputed",
    "time": "11/01/2020 19:44:37"
  },
  {
    "_from": "customers/Zoey",
    "_to": "merchants/Just_Brew_It",
    "amount": 697,
    "status": "Undisputed",
    "time": "10/23/2020 18:49:30"
  },
  {
    "_from": "customers/Zoey",
    "_to": "merchants/Amazon",
    "amount": 421,
    "status": "Undisputed",
    "time": "01/11/2021 20:08:37"
  },
  {
    "_from": "customers/Zoey",
    "_to": "merchants/Subway",
    "amount": 489,
    "status": "Undisputed",
    "time": "11/13/2020 07:46:39"
  },
  {
    "_from": "customers/Ava",
    "_to": "merchants/Sears",
    "amount": 211,
    "status": "Undisputed",
    "time": "12/22/2020 06:38:13"
  },
  {
    "_from": "customers/Ava",
    "_to": "merchants/Walmart",
    "amount": 51,
    "status": "Undisputed",
    "time": "12/15/2020 22:11:41"
  },
  {
    "_from": "customers/Ava",
    "_to": "merchants/American_Apparel",
    "amount": 10,
    "status": "Undisputed",
    "time": "11/07/2020 13:10:23"
  },
  {
    "_from": "customers/Ava",
    "_to": "merchants/American_Apparel",
    "amount": 33,
    "status": "Undisputed",
    "time": "10/23/2020 00:18:40"
  },
  {
    "_from": "customers/Ava",
    "_to": "merchants/Just_Brew_It",
    "amount": 150,
    "status": "Undisputed",
    "time": "10/17/2020 00:04:15"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 957,
    "status": "Undisputed",
    "time": "10/14/2020 00:19:17"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Walmart",
    "amount": 698,
    "status": "Undisputed",
    "time": "12/23/2020 10:49:48"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 649,
    "status": "Undisputed",
    "time": "12/11/2020 16:58:40"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Just_Brew_It",
    "amount": 126,
    "status": "Undisputed",
    "time": "12/21/2020 05:22:57"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 676,
    "status": "Undisputed",
    "time": "01/11/2021 19:48:26"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/Starbucks",
    "amount": 210,
    "status": "Undisputed",
    "time": "11/06/2020 13:12:16"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/McDonalds",
    "amount": 634,
    "status": "Undisputed",
    "time": "01/07/2021 13:12:26"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/Sears",
    "amount": 292,
    "status": "Undisputed",
    "time": "01/23/2021 18:43:13"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/Amazon",
    "amount": 626,
    "status": "Undisputed",
    "time": "12/23/2020 17:06:25"
  },
  {
    "_from": "customers/Mia",
    "_to": "merchants/Soccer_for_the_City",
    "amount": 146,
    "status": "Undisputed",
    "time": "02/07/2021 21:12:46"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/McDonalds",
    "amount": 846,
    "status": "Undisputed",
    "time": "01/30/2021 19:33:52"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Abercrombie",
    "amount": 86,
    "status": "Undisputed",
    "time": "01/27/2021 20:02:35"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Subway",
    "amount": 13,
    "status": "Undisputed",
    "time": "11/01/2020 21:52:46"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Amazon",
    "amount": 46,
    "status": "Undisputed",
    "time": "10/08/2020 15:57:05"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Walmart",
    "amount": 873,
    "status": "Undisputed",
    "time": "12/22/2020 10:09:26"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Apple_Store",
    "amount": 917,
    "status": "Disputed",
    "time": "12/04/2020 21:21:17"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Urban_Outfitters",
    "amount": 858,
    "status": "Disputed",
    "time": "02/02/2021 15:23:20"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/RadioShack",
    "amount": 470,
    "status": "Disputed",
    "time": "01/19/2021 07:00:53"
  },
  {
    "_from": "customers/Paul",
    "_to": "merchants/Macys",
    "amount": 644,
    "status": "Disputed",
    "time": "11/07/2020 08:09:29"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Apple_Store",
    "amount": 446,
    "status": "Disputed",
    "time": "12/29/2020 07:33:43"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Urban_Outfitters",
    "amount": 225,
    "status": "Disputed",
    "time": "01/10/2021 22:06:13"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/RadioShack",
    "amount": 39,
    "status": "Disputed",
    "time": "11/30/2020 17:24:05"
  },
  {
    "_from": "customers/Marc",
    "_to": "merchants/Macys",
    "amount": 928,
    "status": "Disputed",
    "time": "01/27/2021 16:58:42"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Apple_Store",
    "amount": 59,
    "status": "Disputed",
    "time": "12/01/2020 14:22:55"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Urban_Outfitters",
    "amount": 156,
    "status": "Disputed",
    "time": "11/06/2020 02:10:55"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/RadioShack",
    "amount": 85,
    "status": "Disputed",
    "time": "10/28/2020 17:48:22"
  },
  {
    "_from": "customers/Olivia",
    "_to": "merchants/Macys",
    "amount": 437,
    "status": "Disputed",
    "time": "11/16/2020 03:27:31"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Apple_Store",
    "amount": 915,
    "status": "Disputed",
    "time": "12/01/2020 19:07:57"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Urban_Outfitters",
    "amount": 132,
    "status": "Disputed",
    "time": "12/30/2020 08:04:40"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/RadioShack",
    "amount": 881,
    "status": "Disputed",
    "time": "11/27/2020 11:58:56"
  },
  {
    "_from": "customers/Madison",
    "_to": "merchants/Macys",
    "amount": 359,
    "status": "Disputed",
    "time": "12/09/2020 15:01:34"
  }
]

FOR txn IN transactionsData
	INSERT txn IN transactions
  
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
