## Design Redbus 

- Database as they say holds the truth. So, it is quite important that it gets the right respect while architecting and designing applications. At redBus, we use a mixture of SQL, NoSQL databases to solve different kinds of problems. In this blog, we will try to go over some of the generic use cases we are solving and the database of choice.
Some background of redBus to set the context:

This is at a broad level — redBus architecture. As you see there are multitude of systems and we have been generating a whole lot of data at every layer. Application logs, transaction logs, workflow logs .. all these are captured by the individual systems.
Now, let us analyse them:
MySQL — A traditional RDBMS used by tons of organisations. We use this for all our OLTP applications. Payments, Transactions, User Profiles etc.. all are saved in MySQL during the transaction flow. As we are hosted on AWS, we used the RDS variant.
Cassandra — We use this in 2 places
The inventory data we collect from vendors in real time are first saved in our Cassandra system. We cannot afford to lose the data that is stored in this system and we need a highly fault tolerant system. This is one of the main reason we went with Cassandra (more on this from throughput needed etc). It is a columnar store and suits our Data Model precisely as we store a lot of the vendor in real time such as Service Info, Seat Layout, Availability etc..
Our Fraud engine is also running of Cassandra. We make use of an important feature in Cassandra which is Row TTL. It basically ensures there is an automatic deletion of this row post some TTL configuration provided.
3. Google Big Query — We use BigQuery for a bunch of things. Primarily we import our Google Analytics, AdWords data into BigQuery using their connector. This is quite powerful as we have the entire raw data from GA imported in to BigQuery. Now, from other data sources like Transactions etc .. we are able to correlate the data sources as needed by the business.
4. MongoDB — This is currently used as a generic purpose document storage where in specific application logs, configurations etc are stored. We do not do direct analysis on this.
5. REDIS — This is our distributed cache. We make use of the AWS Elastic Cache here. It is one of our primary data stores used in many of our use cases :
Inventory systems stores almost the entire inventory in REDIS. As we need high throughput, this is a good choice.
The payment transactions in transit are stored here for look-up etc.. by various systems. We make use of the pub-sub model of REDIS too to handle lost HTTP calls.
6. CouchBase — We are using this in quite a few use cases. Primarily for its clustering and out of the box API support — it is a strong choice. This is a cusp between MongoDB and REDIS as it supports key-value look up but at the same time, enables querying on the document. The throughput of CouchDB is quite impressive.
7. REDSHIFT — This is our data warehouse plus analytics platform. This houses a ton of data from various data sources — both structured and non-structured. The columnar store is insanely good on performance to crunch data (really large amount) in seconds. Some of the use cases are
Inventory Analysis — we look at the overall Bus Inventory which comprises of close to 1Million seats per DOJ (Date of Journey) which is available. We try to segment this data to get a better understanding of the market.
Seller Analytics — we provide a whole lot of data to our sellers (Bus Operators) so that they understand how they are performing.
There are various pros and cons when we compare REDSHIFT Vs. BigQuery. We will address in a separate blog. Technically for most of our use cases both REDSHIFT and BigQuery are good candidates. Due to certain strategies adopted by vendor (BigQuery <> GA connector by Google, REDSHIFT <> S3, REDSHIFT <> RDS by Amazon), it becomes quite easy to do the necessary ETL and persist the data without much technical overhead.
Apart from these databases, we also use couple of other databases for our m-web and Android applications.
Firebase — This is the latest entrant in our ecosystem. Primarily we are using 2 components of Firebase
Real Time DB — We use this to store our voice notes (support calls) and later on sync with our back-end systems. We are also using to show case some personalised cards (trips) to users based on their profile. These cards are populated by an outside job which observes a ton of other data points and builds the necessary models.
Remote Config — Configuration Management and A/B testing.
2. IndexedDB — As a part of PWA efforts on m-web we store the relevant cache data in IndexDB.
3. SnappyDB — This is a simple key-value datastore. We use this to store a lot of the data that needs to reside on the Apps (Android) for the user like TicketDetails, RecentTrips, ShortListed hotels etc.. The main advantage is it helps in storing the actual document as is from the API signature. Also, upgrades are quite simple when the data model changes.
In the next series of this blog post, we will be doing a deep dive on some of these and provide a better insight in to the mechanics.