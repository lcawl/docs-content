---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/tutorial-manage-new-data-stream.html
applies_to:
  stack: ga
  serverless: ga
products:
  - id: elasticsearch
---

# Creating a data stream with a lifecycle [tutorial-manage-new-data-stream]

This page shows how to configure data stream lifecycle on a new data stream and how to retrieve lifecycle details after creation. For the full setup flow, including component templates, creating the stream, and securing it, refer to [Set up a data stream](/manage-data/data-store/data-streams/set-up-data-stream.md).

1. [Add lifecycle to the index template](#create-index-template-with-lifecycle)
1. [Create the data stream](#create-data-stream-with-lifecycle)
1. [Retrieve lifecycle information](#retrieve-lifecycle-information)

## Add lifecycle to the index template [create-index-template-with-lifecycle]

A data stream requires a matching [index template](../../data-store/templates.md). You configure data stream lifecycle by setting the `lifecycle` field in the index template, the same as you do for mappings and index settings.

Define an index template that sets a lifecycle as follows:

* Include the `data_stream` object to enable data streams.
* Define the lifecycle in the template section or include a composable template that defines the lifecycle.
* Use a priority higher than `200` to avoid collisions with built-in templates. See [Avoid index pattern collisions](../../data-store/templates.md#avoid-index-pattern-collisions).

You can use the [create index template API]({{es-apis}}operation/operation-indices-put-index-template).

```console
PUT _index_template/my-index-template
{
  "index_patterns": ["my-data-stream-test"], <1>
  "data_stream": { },
  "priority": 500,
  "template": {
    "lifecycle": {
      "data_retention": "7d"
    }
  },
  "_meta": {
    "description": "Template with data stream lifecycle"
  }
}
```

1. In this case the index template will be applied to a data stream named `my-data-stream-test`. You can optionally use a wildcard (`*`) in the index pattern to match all data streams created (either manually or using an indexing request) that have a name matching the specified pattern.

For a complete index template example with mappings and lifecycle tabs, refer to [Create an index template](/manage-data/data-store/data-streams/set-up-data-stream.md#create-index-template).

:::{tip}
:applies_to: {"stack": "ga 9.5", "serverless": "unavailable"}

To move older backing indices to the frozen tier automatically, include `frozen_after` in the lifecycle you put on the template. For requirements and how conversion works, refer to [](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md). For example:

```console
PUT _index_template/my-index-template
{
  "index_patterns": ["my-data-stream-test"],
  "data_stream": { },
  "priority": 500,
  "template": {
    "lifecycle": {
      "data_retention": "365d",
      "frozen_after": "30d"
    }
  },
  "_meta": {
    "description": "Template with data stream lifecycle and frozen transitions"
  }
}
```

Confirm that a default snapshot repository is registered before indexing data. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).
:::

## Create the data stream [create-data-stream-with-lifecycle]

After you create an index template with data stream lifecycle, create the data stream using the same steps as any other data stream. Refer to [Create the data stream](/manage-data/data-store/data-streams/set-up-data-stream.md#create-data-stream).

When a write operation with the name of your data stream reaches {{es}}, the data stream is created with the respective data stream lifecycle.

## Retrieve lifecycle information [retrieve-lifecycle-information]

You can use the [get data stream lifecycle API]({{es-apis}}operation/operation-indices-get-data-lifecycle) to see the data stream lifecycle of your data stream and the [explain data stream lifecycle API]({{es-apis}}operation/operation-indices-explain-data-lifecycle) to see the exact state of each backing index.

```console
GET _data_stream/my-data-stream-test/_lifecycle
```

The result will look like this:

```console-result
{
  "data_streams": [
    {
      "name": "my-data-stream-test",                                <1>
      "lifecycle": {
        "enabled": true,                                            <2>
        "data_retention": "7d",                                     <3>
        "effective_retention": "7d",                                <4>
        "retention_determined_by": "data_stream_configuration"
      }
    }
  ],
  "global_retention": {}
}
```

1. The name of your data stream.
2. Shows if the data stream lifecycle is enabled for this data stream.
3. The retention period of the data indexed in this data stream, as configured by the user.
4. The retention period that will be applied by the data stream lifecycle. This means that the data in this data stream will be kept at least for 7 days. After that {{es}} can delete it at its own discretion.

If you want to see more information about how the data stream lifecycle is applied on individual backing indices use the [explain data stream lifecycle API]({{es-apis}}operation/operation-indices-explain-data-lifecycle):

```console
GET .ds-my-data-stream-test/_lifecycle/explain
```

:::{tip}
You can use a wildcard (`*`) in the data stream name to retrieve the lifecycle status for all data streams matching the pattern.
:::

The result will look like this:

```console-result
{
  "indices": {
    ".ds-my-data-stream-test-2023.04.19-000001": {
      "index": ".ds-my-data-stream-test-2023.04.19-000001",      <1>
      "managed_by_lifecycle": true,                               <2>
      "index_creation_date_millis": 1681918009501,
      "time_since_index_creation": "1.6m",                  <3>
      "lifecycle": {                                        <4>
        "enabled": true,
        "data_retention": "7d"
      }
    }
  }
}
```

1. The name of the backing index.
2. If it is managed by the built-in data stream lifecycle.
3. Time since the index was created.
4. The lifecycle configuration that is applied on this backing index.
