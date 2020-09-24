# Relational Database

The relational database contains the secondary data of COLID. There are two main reasons for a secondary database. The first reason is to separate business data from application specific data whereby the application specific data is related to the use cases mentioned in the next section. The second reason is the database system itself: Neptune is a graph database, specialized on storing and querying highly-linked data sets. It is not an optimal infrastructure for (most types of) analytical or operational processes, e.g. serving data for a typical batch process. So an additional database is needed.

This document is intended as a design documentation for the choice made on a secondary database for COLID. It highlights the reasons and the currently known use cases for which a secondary database is needed, which are our options and what is the final decision. 

Hint: Currently the amount of data cannot be estimated but as the business data is inside Neptune, the secondary database will only be responsible for smaller amounts of data.

## Use Cases

We identified the following use cases where a secondary database may come into play:

+ 	**Application configuration**
	In future we'll need the possibility to set and change some configurations at runtime. Our first use case is to display a maintenance screen in the frontend and block the usage of the backend without redeploying a special version. This could be the case in any kind of maintenance (AWS systems, the infrastructure, data migration etc.).
	Also release notes should be stored in the database as they shouldn't be hardcoded or be stored in an external system like a wiki. So it can be changed dynamically.
	
+ 	**Storing the current status of the application and sub components**
	We want to store the current status of the application and it´s sub components, e.g. if a reindexing process is currently running, how old the cache´s content is etc. This could also have some benefits for monitoring of the application.
	
+ **Duplicate checks for PID URIs**
	One requirement of the software is to check wheter a PID URI yet exists if a new COLID entry is set up. Currently this is very inperformant with a graph database and could be more performant with another database type.  

+	**Unique IDs for PID URIs**
	Also when setting up new PID URIs a unique ID is required for every type of URI template.  Currently this is done by counting all graph vertices of a special type and increment this by one. This is oviously not very performant but also not  reliable.
	
+ 	**Check ressources for reachability**
	A new requirement is to check whether the entered resources in the system (e.g. maintenance points) are still available and reachable. Therefore a scheduler is needed to check this frequently. Of course a scheduler (e.g. Hangfire.io) needs some backend to store it´s configuraiton.
	
+	**Storage of user specific data**
	We are going to store user specific data in future. This could be a user specific configuration, e.g. standard values when entering a new COLID entry. It also could be a special set of search parameters or even different versions of search requests. A further use case could be the implementation of mailing alerts for special events, e.g. a COLID entry is modified or a new version is published.

## Options

This section should highlight three candidates to become a secondary database. As we´re relying on the infrastructure of the AWS cloud, we´ll only have a look on AWS managed services. The benefits are  the automatic failover, backup and recovery processes.

To have a comparable view to these candidates, we´ll take the following 'criteria' into account. Of course there are some more critera but these seems to be a good subset to us:

+ Introduction
+ ACID-Compliance
+ Performance
+ Adaptability
+ Queryability
+ Indexing
+ Costs

As costs are dependent on different factors for each db type, there is no meaningful comparison possible at this time.

### Amazon RDS / Aurora

Amazon RDS is a management system for different types of databases. In this case we´ll have a concrete view on Aurora, which is a Amazon owned traditional SQL database which is compatible with well known databases like MySQL, PostgreSQL and others.

**ACID**: Fully supported

**Performance**: Traditional use cases where one needs to store a lot of similar structured data, e.g. like orders and their current status. The data is mostly related to each other.

**Adaptability**: As known from any other SQL database the schema can easily be changed, e.g. by adding a new column.

**Queryability**: Standard SQL commands are supported.

**Indexing**: Indices can be set as known from other SQL databases.

**Costs**: -

### Amazon DynamoDB

DynamoDB is a special kind of key-value and document-oriented database. It combines different primitive values but also documents with one key and it is mostly used for high scaling internet applications, real-time offerings and customer specific data.

**ACID**: Yes, when explicitely using transactions. Else only Consistency / Durability.

**Performance**: Optimized for storing/working on incomplete data sets according to the logic schema. Strong read/write consistency is very expensive.

**Adaptability**: It is given in general but it´s recommended to design the database earliest when all requests are known.

**Queryability**: Querying is done by using  json style documents. 

**Indexing**: One or more secondary indices can be created for any column. The index will result in an additional table which is used internally but which also can be queried seperately.

**Costs**: - [no small dev systems available, only memory optimized ones]

### Amazon DocumentDB

DocumentDB is a document oriented database from Amazon which is compatible with MongoDB (3.6) and is often used for content management, personalisation and mobile applications.

**ACID**: Not clearly stated by Amazon.

**Performance**: Optimized for document based storage and analytics without a direct relation between the documents.

**Adaptability**: it is given in general but might be an expensive task as all documents inside a collection needs to be updated.

**Queryability**: Querying is done by using  json style documents. 

**Indexing**: Indices can be created on any field. Arrays will be indexed as one entry and only if < 2 MB. The usage of an index must be given in addation to every query

**Costs**: - [only two small dev systems available beside memory optimized ones]

## Descision

With respect to the current known use cases and the assumed data, amount of data and data structure it is currently the easiest way to use Amazon RDS/Aurora. Also it´s principles (schemas, transactions, query language, ...) are well known to the developers, the entity framework provides a fast and easy access and also standard sequences can be used to generate IDs for the PID URIs.