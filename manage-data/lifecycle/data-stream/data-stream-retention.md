---
navigation_title: Data stream retention
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/tutorial-manage-data-stream-retention.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: elasticsearch
description: Learn how data stream lifecycle retention works, including data stream, global default, and global max retention settings.
type: overview
---

# Data stream retention [data-stream-retention]

Data stream lifecycles define the minimum time {{es}} keeps data in a data stream.
After that period, {{es}} can delete older backing indices to free space and manage storage costs.
This page explains the types of retention settings, how {{es}} calculates the effective retention for each data stream, and when deletion runs during a lifecycle pass.

:::{note}
This information applies only to data streams managed by [data stream lifecycles](/manage-data/lifecycle/data-stream.md).
:::

There are four different types of retention:

* The data stream retention (`data_retention`), which configured on the data stream level. It can be set using an [index template](../../data-store/templates.md) for future data streams or using the [PUT data stream lifecycle API]({{es-apis}}operation/operation-indices-put-data-lifecycle) for an existing data stream. When the data stream retention is not set, it implies that the data must be kept forever.
* The global default retention (`default_retention`), which is configured through the cluster setting [`data_streams.lifecycle.retention.default`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#data-streams-lifecycle-retention-default) and applied to all data streams managed by data stream lifecycle that do not have `data_retention` configured. Effectively, it ensures that there will be no data streams keeping their data forever. This can be set using the [update cluster settings API]({{es-apis}}operation/operation-cluster-put-settings).
* The global max retention (`max_retention`), which is configured through the cluster setting [`data_streams.lifecycle.retention.max`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#data-streams-lifecycle-retention-max) and applied to all data streams managed by data stream lifecycle. Effectively, it ensures that there will be no data streams whose retention exceeds this time period. It can be set using the [update cluster settings API]({{es-apis}}operation/operation-cluster-put-settings).
* The effective retention (`effective_retention`), which is applied to a data stream on a given moment. Effective retention cannot be set, it is derived by taking into account all the preceding retention settings and is [calculated](#effective-retention-calculation).

::::{note}
Global default and max retention settings do not apply to data streams internal to Elastic. Internal data streams are recognized either by having the `system` flag set to `true` or a name that is prefixed with a dot (`.`).
::::

## How is the effective retention calculated? [effective-retention-calculation]

The effective is calculated in the following way:

* When `default_retention` is defined and the data stream does not have `data_retention`, the `effective_retention` is the `default_retention`.
* When `data_retention` is defined and it is less than the `max_retention` (if it's defined), the `effective_retention` is the `data_retention`.
* When the data stream has either no `data_retention` or its `data_retention` is greater than the `max_retention` (and `max_retention` is defined), the `effective_retention` is the `max_retention`.

This behavior is demonstrated in the following examples:

| `default_retention` | `max_retention` | `data_retention` | `effective_retention` | Retention determined by |
| --- | --- | --- | --- | --- |
| Not set | Not set | Not set | Infinite | N/A |
| Not relevant | 12 months | 30 days | 30 days | `data_retention` |
| Not relevant | Not set | 30 days | 30 days | `data_retention` |
| 30 days | 12 months | Not set | 30 days | `default_retention` |
| 30 days | 30 days | Not set | 30 days | `default_retention` |
| Not relevant | 30 days | 12 months | 30 days | `max_retention` |
| Not set | 30 days | Not set | 30 days | `max_retention` |

You can view these details with the [get data stream lifecycle API]({{es-apis}}operation/operation-indices-get-data-lifecycle):

```console
GET _data_stream/my-data-stream/_lifecycle
```

Retention settings appear in the response like this:

```console-result
{
  "global_retention" : {
    "max_retention" : "90d",                                   <1>
    "default_retention" : "7d"                                 <2>
  },
  "data_streams": [
    {
      "name": "my-data-stream",
      "lifecycle": {
        "enabled": true,
        "data_retention": "30d",                                <3>
        "effective_retention": "30d",                           <4>
        "retention_determined_by": "data_stream_configuration"  <5>
      }
    }
  ]
}
```

1. The maximum retention configured in the cluster.
2. The default retention configured in the cluster.
3. The requested retention for this data stream.
4. The retention that is applied by the data stream lifecycle on this data stream.
5. The configuration that determined the effective retention. In this case it's the `data_configuration` because it is less than the `max_retention`.

## How is the effective retention applied? [effective-retention-application]

Retention is applied to the remaining backing indices of a data stream as the last step of [a data stream lifecycle run](../data-stream.md#data-streams-lifecycle-how-it-works). Data stream lifecycle will retrieve the backing indices whose `generation_time` is longer than the effective retention period and delete them. The `generation_time` is only applicable to rolled over backing indices and it is either the time since the backing index got rolled over, or the time optionally configured using the [`index.lifecycle.origination_date`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#index-data-stream-lifecycle-origination-date) setting.

::::{important}
The `generation_time` is used instead of the creation time because this ensures that all data in the backing index has passed the retention period. As a result, the retention period is not the exact time data is deleted. Rather it is the minimum time data is stored.
::::

## Next steps

Configure retention for your data streams:

* New data streams: set `data_retention` in an index template when you [create a data stream with a lifecycle](/manage-data/lifecycle/data-stream/tutorial-create-data-stream-with-lifecycle.md).
* Existing data streams: update `data_retention` with the [update data stream lifecycle tutorial](/manage-data/lifecycle/data-stream/tutorial-update-existing-data-stream.md).

## Related pages

* [Data stream lifecycle](/manage-data/lifecycle/data-stream.md)
* [Data stream lifecycle settings](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md)
