# Logging

To monitor the use of the application, all services of COLID Editor and Data Marketplace generate log entries via API interfaces and functions used within the applications.
Included is which person called which application via which interface at what time with which HTTP function.
According to the GDPR, a nightly anonymizer job within the Scheduler Service takes care of anonymizing the logged personal data of the previous day.
All personal data within the log entries that are in the Elasticache are hashed with the help of a predefined anonymizer key, so that it is no longer possible to trace which person generated which log entries. 
Only the application ID is retained to allow conclusions to be drawn about an individual application (e.g. a call from the COLID frontend to the COLID Registration Service).
Through this anonymization procedure it is possible to make a statement about the number of users and applications using COLID and Data Marketplace (see [here](infrastructure/databases/elasticsearch/application_insights/)).
Finally, after a retention period of one year, the personal data is completely anonymized, so that the statement about the number of users is no longer possible and only the number of accesses across all users can be investigated.