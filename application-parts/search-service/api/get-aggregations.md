# Get Aggregations

Getting the initial aggregations helps the user to further deep dive into the data and to see how many COLID entries for the defined aggregations are available. Those aggregation can be used to filter the search query.

## API Usage

```shell
POST "https://localhost:51800/api/search/aggregations" 
```

## Description

Aggregations can be used to build complex summaries of the data. Those units build analytic information over a set of documents and can be used to filter down to narrower information in the data.

The above API endpoint is used at the Data Marketplace welcome page to return all initial aggregations. Which fields of a document will be used as aggregation is defined the current metadata set. The API endpoint returns all available aggregations for the defined fields in the index, this is why no parameters are needed. 