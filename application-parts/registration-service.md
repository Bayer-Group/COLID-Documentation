# Registration Service
The Registration Service is the core of the COLID application. The data-centric service receives the incoming information from the COLID Editor or other API application through a defined REST interface in JSON format, validates the incoming data against predefined metadata as well as SHACLs and finally stores the information as so-called "COLID entries" in the graph database in RDF format. The metadata and SHACLs are also available to the Registration Service in RDF format, which it makes available to users via the API.

## Technology Stack

1. ASP.NET Core 3.1
1. Docker image for deployment

### Special libraries

- [dotNetRDF](https://github.com/dotnetrdf)

## Application Architecture

![Architecture of Registration Service](registration-service/assets/registration_architecture-overview.svg)

## Communication

- Gets called by COLID Editor client via REST-API to create/edit/delete COLID entries, consumer groups, and other entities to the graph
- Gets called by 3rd-party API services via REST-API to create/edit/delete COLID entries
- Calls REST-API AppData Service to create consumer groups
- Calls REST-API of Indexing Crawler Service to create new index
- Pushes data to Message Queue for Search Service and to index COLID entries
- Interacts with Graph Database via SPARQL HTTP-interface for CRUD actions of entities
