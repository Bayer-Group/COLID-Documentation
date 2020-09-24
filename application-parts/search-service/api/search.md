# Search

This is the centerpiece of the Data Marketplace search service. With this API call all published COLID entries can searched and aggregated as defined in the search parameters. 

## API Usage

```shell
POST "https://localhost:51800/api/search" 
{
  "searchTerm": "string",
  "from": 0,
  "size": 0,
  "aggregationFilters": {
    "additionalProp1": [
      "string"
    ],
    "additionalProp2": [
      "string"
    ],
    "additionalProp3": [
      "string"
    ]
  },
  "rangeFilters": {
    "additionalProp1": {
      "additionalProp1": "string",
      "additionalProp2": "string",
      "additionalProp3": "string"
    },
    "additionalProp2": {
      "additionalProp1": "string",
      "additionalProp2": "string",
      "additionalProp3": "string"
    },
    "additionalProp3": {
      "additionalProp1": "string",
      "additionalProp2": "string",
      "additionalProp3": "string"
    }
  },
  "noAutoCorrect": true,
  "apiCallTime": "string"
}
```

## Description

This API endpoint enables the users to use the search functionality provided by the search service to find COLID entries. The algorithms used for search are designed to provide the user with the most relevant COLID entries for their search query. Besides the search there is also the functionality to aggregate the data using different dimensions.

