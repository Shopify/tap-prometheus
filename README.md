> **This repository is no longer maintained.** It has been archived by Shopify and receives no updates or security fixes.

# tap-prometheus

This is a [Singer](https://singer.io) tap that produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

Module loads PromQL query result and pushes the result as a record.

Sample config to calculate daily online peak by customer and environment:

```$json
{
  "endpoint": "http://localhost:9000",
  "auth": {
    "username": "prometheus",
    "password": "123"
  },
  "start_date": "2018-11-01T00:00:00Z",
  "metrics": [
    {
      "name": "online_peak",
      "query": "sum(sessions_count)",
      "batch": "10000",
      "step": "120s", 
      "labels": {
        "type": "object",
        "properties": {
          "customer": {
            "type": "string"
          },
          "environment": {
            "type": "string"
          }
        }
      }
    }
  ]
}
```

The top-level configuration settings for this tap are

* `endpoint` *required*: The Prometheus API endpoint URL - do not include the obligatory `/api/v1` suffix in here, the tap will add this itself
* `auth` *optional*: HTTP basic auth `username` and `password`
* `start_date` *required*: Earliest date from which to start collecting data
* `metrics` *required*: Definition of individual prometheus metric query streams
  
This tap emits a stream for every `metric` definition.

* `query`: PromQL query, must be a range query
* `batch`: The maximum number of steps to move forward in the stream in one batch
* `step`: metrics resolution in seconds
* `labels`: JSON Schema for labels returned by the query. You can set this to `{"type": "null"}` if your query does not return any labels.

You can tune `batch` size to your prometheus instance and singer tap memory requirements. For typical setups it should be fine to fetch up to 10.000 data points per prometheus API query.

Several source code parts copied from:

* tap-stripe: https://github.com/singer-io/tap-stripe

Module based on the patched python Prometheus API client *promalyze*: https://github.com/meshcloud/promalyze

## Capabilities

This tap supports incremental replication. It will run collection until the current extraction time rounded down to the last full step (see configuration).
If you're using tap with meltano, configure it with the following capabilities

```yml
capabilities:
- catalog
- discover
- state
```
