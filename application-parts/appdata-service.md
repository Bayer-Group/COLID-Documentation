# AppData Service

This design document will give an overview how application specific data will be handled by COLID. The definition of application specific data in this context covers all data that is only used within the application itself and should not be available for other systems, even with API calls. So there is a clear differentiation to application business data which might also be of interest for other applications/systems.

The use cases for application specific data are the followings:

- Store user data and user configurations regarding the Editor/Data Marketplace system.
- Store and apply subscriptions/stored queries of users to COLID entries.
- Store and send notifications to the user regarding their subscriptions/stored queries.
- Generation of unique IDs for URI templates.

Our solution make use of a secondary database which will be a SQL database. So there is a clear differentiation of the business data and the app specific data. Furthermore the services to access the database including the needed logic will be separated into it´s own web service.
Because most of the end users have a preset private mode in their browsers cookies are no option.

## Technology Stack

1. ASP.NET Core 3.1
1. Entity Framework Core 3.1.3
1. Microsoft Graph 3.7.0
1. MySQL database
1. Docker image for deployment

### External libraries

- [Pomelo.EntityFrameworkCore.MySql](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)
- [AutoMapper](https://automapper.org/)

## Application Architecture

![Architecture of AppData Service](appdata-service/assets/appdata_architecture-overview.svg)

## Communication

- Gets called by COLID Editor frontend and Data Marketplace while saving default consumer group
- Gets called from Registration Service while saving Consumer Groups in Registration Service
- Get sequentially called by Scheduler Service from running background jobs
- Interacts with relation database

## Data Model

The following diagram describs the structure of the database for the named use cases and will be desribed in the following paragraphs. In general every table has a *created_at* and *modified_at* column, which is best practices if it comes to data analysis triggered by raised problems.

The single texts right to the middle tables are enum values which should be used as column values but without creating an additional table where these 'keys' are defined centrally.

![](appdata-service/assets/appdata_entity-relation-diagram.svg)

**Table 'Users'**

This tables stores the main user data which is the *Id* of the user (Guid) and it´s *email address*. In addition the last login time for Editor/Data Marketplace will be stored for analytics. The *last_time_checked* column will store the information when the user was checked against Azure AD whether the user still exist or the email address has changed. There is also a field called *DefaultSearchFilterDataMarketplace*, which contains the *id* of a search filter for Data Marketplace.

**Table 'Consumer_Groups'**

This table stores different consumer groups, that can be individually choosen per user as a preset value when accessing COLID. Instead of the name, the identifier of the consumer group (URI) will be stored.

**Table 'Search_Filter_Editor'**

This table stores the search filter, that a singled user has used. The requirement states that only the last used filter configuration will be stored. This configuration will be stored as a JSON object (which can automatically de-/serialized from/by the currently used DTO ).

**Table 'Search_Filters_Data_Marketplace'**

This table is like the *Search_Filters_Editor* but can store several filter configurations. Therefore the user can input a *name* per configuration shich should be stored.

**Table 'Stored_Queries'**

This table stores search queries a user submitted in Data Marketplace. Therefore the query gets a *name*, the query itself in *JSON* format and an *execution interval* in which the query execution time will be automatically determined by the system. The exact execution time will be written in the *NextExecutionAt* field. The *last result* of such an automatic execution will also be stored to find the differences between two runs.

**Table 'COLID_Entry_Subscriptions'**

This table stores the COLID entries which users has subscribed to. Therefore all the COLID entries (pid uris) are stored in combination with the users unique Guid.

**Table 'Messages'**

This tables stores all messages a user gets over time. The message itself is splitted into an *id*, a *subject* and *body*. Also several meta information like the *read date* of the message per user or when the message should be *send to* him via email is set in the fields with the same name. For extending logic, a *delete on* field indicated the time, when the message shall be deleted.

**Table 'Messages_Templates'**

This table stores templates for messages, created by an administrator. The *message type* describes e.g. a changed resource or a new result from a stored query. Therefore the *type* of the message - e.g. a changed resource or a new result from a stored query - will identify the valid template. Every template consists out of an *id*, a *subject* and a *body*. This entity is not linked to messages, because the purpose is to copy the content and replace all placeholders, that has been applied to the template's message subject and/or body.

Hint: Currently there are no typical *valid_from* and *valid_to* columns.

**Table 'Message_Configs'**

This tables stores information of each user when unread messages  (in Editor or Data Marketplace) will be sent or *sent* messages (e.g. by mail) will be deleted.

**Table 'Welcome_Message'**

The landingpage of Editor or DataMarketplace contains a welcome message to every user with several custom information. These kind of informations are stored within this table. The *ColidType* is marked as unique via constaint and can be added only once.

---

**[WIP] WIP: Table 'Sequences'**

This table is a handy workaround as MySql does not support typical out-of-the-box sequences. So this table will simulate sequences whereby it´s possible to dynamically create as much sequences as needed. In general these sequences will be created per URI template so every sequence can act as a unique id creator per such a template. Therefore the URI templates ID is the *name* of such a sequence. The other fields are describing a typical sequence structure.

**[WIP] Table 'URI_Template_IDs'**

This tables stores for every URI template the *URIs* which are making use of that template and also the generated ID from a *sequence* (see above). The template here is represented by the sequence ID.

This concept makes it possible to have a fast overview of all URIs using a template but also to reuse ID which were created in the past but abbandoned, e.g. a URI was deleted.

The *semaphore* will be used to block IDs for a short period of time. So parallelity shouldn´t be a problem when parallel callers require an ID at nearly the same time.

**[WIP] Stored Procedure 'getAndBlockUriTemplateIdBySequenceName'**

This stored procedure will make use of the two above described tables. It is called with the name of a  *sequence* (so the URI template ID) and a *semaphore* (typically a UUID generated just-in-time by the calling thread).  With these to values the SP can block a yet existing but abandoned ID but also request a new ID if necessary. If a new ID was requested the ID will directly be inserted and blocked with the semaphore in the *URI_Template_IDs* table.

The return value is the *sequence ID* which was blocked for the caller.


