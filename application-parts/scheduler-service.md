# Scheduler Service

The *Scheduler Service* is, as the name suggests, a service to schedule and execute different kinds of regularly running jobs. The service is implemented with the *Hangfire* library which is used to create, process and manage the several jobs.


## Technology Stack

1. ASP.NET Core 3.1
1. Docker image for deployment

### Special libraries

- [Hangfire Background Scheduler [Open Edition]](https://www.hangfire.io/)


## Application Architecture

![Architecture of the Scheduler-Service](scheduler-service/assets/20200615-Scheduler-Architecture.svg)


## Communication

- Calls REST-API of AppData Service to get scheduler tasks
- Interacts with relation database for Hangfire Background Scheduler database

## Data Model

The data model is depending on the hangfire library and handled by the library itself.
