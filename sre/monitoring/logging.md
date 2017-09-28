Logging
=====

## Strategy

Cloud virtual machines cannot be trusted to maintain state or data, hence, there's a common pattern to expect ephemerality and thus to design for immutability. Whatever logging solution you may choose it should be well planned. Here's some considerations:

* The data should be queriable externally AND internally
* Data should be able to be well organized and presented cleanly
* Alerting should be possible but not strictly relied upon
  * Logging systems are great but often logs indicate that a problem has already occurred
  * Alerting is often not a logging systems core competency. They're typically playing catch up parsing all that data.
* Logs should never exist on a server for more than a few seconds

## Tools


### [Splunk](https://www.splunk.com/)

Type: SaaS/On-Prem
License: Open Source/Subscription

**Pros**

* Cheaper than the alernatives
* Advanced query language
* Alerting
* Fast processing
* Accepts a large array of data types

**Cons**

### [Sumo Logic](https://www.sumologic.com/)

Type: SaaS
License: Subscription

**Pros**

* Advanced query language
* Advanced analytics and dashboarding
* Alerting
* Accepts a large array of data types

**Cons**

* Most expensive

### [Elastic Search/Logstache/Kibana (ELK Stack)](https://www.elastic.co/products)

Type: SaaS/On-Prem
License: Open Source/Subscription

**Pros**

* Open Source
* Extendable
* Accepts a large array of data types

**Cons**

* Poor querying
* Poor dashboarding
* Resource intensive
