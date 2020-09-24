# Delete An Entry

This section explains how to delete an entry in COLID by using the API.

## API Usage

```shell
DELETE "https://localhost:51770/api/v3/resource/markForDeletion?pidUri=<...>"
```

## Description

In fact only an administrator is able to delete an entry physically. But you can mark an entry for deletion so it won´t show up in any search results but also an administrator will view the request for deletion and is able to proceed.

To mark an entry for deletion only the `resource/markForDeletion` interface needs to be called with the specific PID URI.