# Jobs

The following jobs are implemented in this service:

- **Data Quality**
    - User validity check - A job to determine the validity (active or inactive) of given users.

- **Message Handling**
    - MessageMailing - Job to mail all still unread messages (as a collection) per user in a user defined time interval.
    - MessageDeletion -  Job to delete messages per user after a user defined time span.

- **StoredQueryExecution** - Job to execute stored queries per user regularly via the Search Service.