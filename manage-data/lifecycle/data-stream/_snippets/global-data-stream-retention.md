Optionally set cluster-wide default and maximum retention for all data streams managed by data stream lifecycle.
Use these settings to ensure no data stream keeps data indefinitely and to cap how long any data stream can retain data.

Use the [update cluster settings API]({{es-apis}}operation/operation-cluster-put-settings) to set one or both cluster settings:

* [`data_streams.lifecycle.retention.default`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#data-streams-lifecycle-retention-default) — applied to data streams that do not have `data_retention` configured.
* [`data_streams.lifecycle.retention.max`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#data-streams-lifecycle-retention-max) — caps the effective retention for all managed data streams.

```console
PUT /_cluster/settings
{
  "persistent" : {
    "data_streams.lifecycle.retention.default" : "7d",
    "data_streams.lifecycle.retention.max" : "90d"
  }
}
```

For more details, refer to [Data stream retention](/manage-data/lifecycle/data-stream/data-stream-retention.md).
