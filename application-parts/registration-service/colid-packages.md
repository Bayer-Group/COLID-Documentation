# COLID packages

COLID consists of several microservices that share individual components and functionalities. Therefore, different functionalities have been separated from the individual services and divided into reusable modular packages. For this purpose, COLID provides several packages that differ technically and in their functionalities. These packages are maintained in the Registration Service repository.

Since COLID uses several database types that have a decisive influence on the structure of the packages, they are structured in such a way that there is one graph-specific package and additional packages that are used by all services.

## COLID.Common

The COLID.Common package provides all useful and frequently used basic functions and methods. This includes simple guards, extensions and helper classes.

## COLID.Exception

The COLID.Exception package is the baseline for the uniform exception handling of all services. The package includes a middleware that processes errors in requests and responses and returns a uniform format across all services to the clients. Logging of errors is also part of the package. 

## COLID.Graph

The COLID.Graph package provides all necessary functionality to access COLID specific data on the graph database.

Therefore it is divided into three namespaces. 

* The *triplestore* namespace is the connection between the service and the database.  
* The *metadata* namespace provides the functionality to retrieve the metadata of different entity types from COLID. 

## COLID.Identity

The COLID.Identity package includes all authorization and authentication functionalities that ensure a secure operation of the services. It is based on the Azure Active Directory. Helpful basic functionalities are provided to handle the authentication between services. 

## COLID.Maintenance

The COLID.Maintenance package provides functions to put the corresponding microservice into a maintenance mode. For this purpose a flag is set by cache, which is checked at every new request to an interface. When activated, only the functionality of the interface to cancel the mode can be reached. Otherwise a status code 503 (Service Unavailable) is returned. 

### TODO

- Remove IGeneralLogService
- Increase the caching expiration time

## COLID.Swagger

The COLID.Swagger package provides the functionality to easily add the swagger documentation to the service according to the standards defined in COLID. A versioning of the api is required. 

### TODO

* Adapt to generic approach for usage in multiple services
* Adapt version handling - [See](https://stackoverflow.com/questions/26733/getting-all-types-that-implement-an-interface)

## COLID.Cache 

The COLID.Cache package provides the functionality to use the cache used in COLID. Implementations for inmemory and distributed caching are provided, as well as functions for distributed resource locking.   

## COLID.StatisticsLog

The COLID.StatisticsLog package provides the functionality for uniform logging in COLID. It is classified into audit trail, performance and statistical logs. 

### TODO

* Remove GeneralLogService - use ILogger in all services instead

## COLID.MessageQueue

The Colid.MessageQueue package provides the functionality to send and receive messages to other services with the message queue. 

