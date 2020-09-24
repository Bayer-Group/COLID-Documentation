# Error Handling

This document covers the topic of error handling for the COLID / Data Marketplace systems of the WebApi and Services and also the UI of COLID / Data Marketplace. It doesn't cover errors nor the mechanisms of 3rd party systems.

As it is hard to handle all kinds of exceptions with a generic approach, we´ll categorize the exceptions into the following two classes:
+ *TechnicalException*
This are all kind of exceptions which are based on technical problems. This could be e.g. a not rechable web service, a http error code from another system or also too less disk space for storing files.
+ *BusinessException*
This are all kind o exceptions which are based on exceptions due to business logic. This could be e.g. an invalid data validation result, insufficient data given to the API or also unhandles cases in algorithms.

These both exceptions will derive from a *GenericException* which is also a kind of a fallback mechanism, e.g. in case of an unexpected exception from a 3rd party library. In addition the exceptions can be refined in sub classes if necessary.

The *GenericException* will take the following common fields:

```json
{
	"Type": "EntityNotFound",
	"Code": 404,
	"Message": "Your dataset could not be found.",
	"RequestId": "12345"
}
```
Any exception that cannot be handled by COLID is converted to the general exception and returned to the user as "Internal Server Error". 

All exceptions will be handled by a so-called middleware which will be implemented within the WebApi and the UI:

+ .Net / WebApi - [Description](https://docs.microsoft.com/de-de/aspnet/core/fundamentals/middleware/?view=aspnetcore-3.1) / [Example](https://blogs.msdn.microsoft.com/brandonh/2017/07/31/using-middleware-to-trap-exceptions-in-asp-net-core/)
+ Angular / UI - [Description](https://angular.io/api/core/ErrorHandler) / [Example](https://medium.com/@amcdnl/global-error-handling-with-angular2-6b992bdfb59c)

### WebApi / Backend

All exceptions which occure somewhere in the backend should be catched and transformed to one of the exception types above. String values should be centralized in a constants class. So we´ll also have a good overview of possible and handled issues. The transformed exception will be handled by the middleware where it will be logged and sent back to the requester.

### UI / Frontend

All exceptions which occure somewhere in the frontend should be catched and transformed to one of the exception types above. String values should be centralized in a constants class. So we´ll also have a good overview of possible and handled issues. The transformed exception will be handled by the middleware where it will be logged and displayed to the user in a standard dialogue. 

Exceptions from the backend will also be handled by the middleware in the described manner.

## Implementation details

Currently the errors used in COLID have the following structure. Each additional level has additional fields included that describe the specific bug.

+ GeneralException
  + TechnicalException
    + DatabaseException
    + TransactionNotComittedException
  + BusinessException
    + ValidationExeption
      + EntityValidationException
      + ResourceValidationException
    + EntityNotFoundException
    + InvalidFormatException
    + MissingParamterException
    + ReferenceException
    + RequestException

## Todo
+ Backend 
  + Exceptions overview (short)
  + Middleware
  + DTOs
+ Frontend
  + Exceptions overview (short)
  + Middleware
  + DTOs
  + Handling of backend exception objects