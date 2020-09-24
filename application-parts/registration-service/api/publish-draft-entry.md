# Publish A Draft Entry

This section explains how to publish a draft entry in COLID by using the API.

## API Usage

```shell
PUT "https://localhost:51770/api/v3/resource/publish?pidUri=<...>"
```

## Description

To publish a draft entry only the `resource/publish` interface needs to be called with the draft´s specific PID URI. So the draft will become the new published entry while the 'old' one will become a historic version.
