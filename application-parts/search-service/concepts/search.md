# Search and Aggregations

The provided search functionality in Data Marketplace is using Elasticsearch as underlying search engine. All queries and responses will be handled by the Data Marketplace search service, which means all logic for communication with Elasticsearch is implemented here. Further details on searching, filtering and sidebar aggregations are listed below.

## Search

### Query string query

The search bar in Data Marketplace uses the so called, `query_string` query. This query provides full text queries and understand how the fields being queried is analyzed and will apply the right `analyzers` or `search_analyzer` to the query string before executing. Additionally this type of query supports also the compact Lucene query string syntax (like AND/OR conditions).

The search query will be executed against all fields defined in the index mapping.

### Search Hit Highlighting
Highlighters enable you to get highlighted snippets from one or more fields in your search results so you can see where the query matches are. The response contains an additional highlight element for each search hit that includes the highlighted fields and the highlighted fragments.

Highlighting requires the actual content of a field. Therefore, the actual `_source` is loaded and the relevant field is extracted from `_source`.

### Filters

The following types of queries are used in the filter context:

- Term query
- Data range query
- Match query

## Sidebar aggregations
