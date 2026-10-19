---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/tutorial-manage-existing-data-stream.html
applies_to:
  stack: ga
  serverless: ga
description: Step-by-step tutorial for updating the lifecycle of an existing Elasticsearch data stream. Change retention, move older data to the frozen tier, or remove the lifecycle.
type: tutorial
products:
  - id: elasticsearch
---

# Update the lifecycle of a data stream [tutorial-manage-existing-data-stream]

Change how long an existing {{es}} data stream keeps its data, or remove that lifecycle. This tutorial is for you if the stream already exists and you want to update that one stream, not the index template that created it.

By the end of this tutorial, you can:

* Set or change how long a data stream keeps data.
* {applies_to}`stack: ga 9.5+` {applies_to}`serverless: unavailable` Move older [backing indices](/manage-data/data-store/data-streams.md#backing-indices) to the frozen tier.
* Remove the data stream lifecycle.
* Check the retention in {{kib}} or with the explain API.

These steps are for data stream lifecycle only. To configure {{ilm}}, refer to [{{ilm-init}}](/manage-data/lifecycle/index-lifecycle-management.md). For a comparison between the two, refer to [](/manage-data/lifecycle.md).

## Before you begin [update-data-stream-lifecycle-prereqs]

To update the data stream in this tutorial, you need:

* An existing data stream. To create one, refer to [Create a data stream with a lifecycle](/manage-data/lifecycle/data-stream/tutorial-create-data-stream-with-lifecycle.md).
* Access to [{{dev-tools-app}} Console](/explore-analyze/query-filter/tools/console.md) in {{kib}}, or another way to send {{es}} API requests.
* These [index privileges](elasticsearch://reference/elasticsearch/security-privileges.md#privileges-list-indices) on the data stream name. Privileges on a data stream also apply to its [backing indices](/manage-data/data-store/data-streams.md#backing-indices). Refer to [Granting privileges for data streams and aliases](/deploy-manage/users-roles/cluster-or-deployment-auth/granting-privileges-for-data-streams-aliases.md).
    * `manage_data_stream_lifecycle`, to update or remove the lifecycle. This privilege also covers the explain API.
    * `view_index_metadata`, if you only read lifecycle state with the explain API.
* {applies_to}`stack: preview =9.1, ga 9.2+` {applies_to}`serverless: ga` The **All** privilege for the **Streams** feature in {{kib}}, if you change retention in [Streams](/solutions/observability/streams/streams.md).
* {applies_to}`stack: ga 9.5+` {applies_to}`serverless: unavailable` A default snapshot repository, if you move older backing indices to the frozen tier. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).

For an overview of how a data stream lifecycle stores and deletes data, refer to [](/manage-data/lifecycle/data-stream.md).

## Set a data stream's lifecycle [set-lifecycle]

Add or change the retention period with the **{{index-manage-app}}** tools in {{kib}} or the {{es}} [lifecycle API]({{es-apis}}operation/operation-indices-put-data-lifecycle).

:::::{tab-set}
:group: kibana-api
:::{tab-item} {{kib}}
:sync: kibana

To change the data retention settings for a data stream:

1. Find **{{index-manage-app}}** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Data Streams** tab.
1. Search for the data stream.
1. Select the data stream to view its details.
1. In the flyout, select **Manage → Edit data retention**. You can do any of the following:

    - Select how long to retain your data, in days, hours, minutes, or seconds.
    - Select **Keep data indefinitely**. The data stream stays managed but {{es}} does not delete the data. Managing a {{tsds}}, such as logs or metrics, helps {{es}} store your data even when you don't set a retention period.
    - Turn off **Enable data retention** to stop data stream lifecycle management for this data stream.

    If [{{ilm-init}}](/manage-data/lifecycle/index-lifecycle-management.md) already manages the data stream, edit the associated {{ilm-init}} policy instead.
:::
:::{tab-item} API
:sync: api

To change the data retention settings for a data stream:

* Set an indefinite retention period so {{es}} does not delete the data. For example:

    ```console
    PUT _data_stream/my-data-stream/_lifecycle
    { } <1>
    ```

    1. An empty payload means that your data stream is managed but {{es}} does not delete the data. Managing a {{tsds}}, such as logs or metrics, helps {{es}} store your data even when you don't set a retention period.

* Or set a retention period. For example:

    ```console
    PUT _data_stream/my-data-stream/_lifecycle
    {
      "data_retention": "30d" <1>
    }
    ```

    1. The retention period of this data stream is 30 days. {{es}} can delete data older than 30 days.
:::
:::::

The lifecycle change applies to every backing index of the data stream.

Check the change in {{kib}} or with the {{es}} [explain API]({{es-apis}}operation/operation-indices-explain-data-lifecycle):

:::::{tab-set}
:group: kibana-api
:::{tab-item} {{kib}}
:sync: kibana

To check the data retention settings for a data stream:

1. Find **{{index-manage-app}}** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Data Streams** tab.
1. Search for the data stream.
1. Select the data stream to view its details. The flyout shows the data retention settings. If an [{{ilm-init}} policy](/manage-data/lifecycle/index-lifecycle-management.md) manages the data stream, **Effective data retention** can differ from **Data retention**.

  :::{image} /manage-data/images/elasticsearch-reference-lifecycle-status.png
  :alt: Index lifecycle status page
  :width: 500px
  :::
:::
:::{tab-item} API
:sync: api

To check the data retention settings for a data stream:

```console
GET .ds-my-data-stream-*/_lifecycle/explain
```

**Result:**

```console-result
{
  "indices": {
    ".ds-my-data-stream-2023.04.19-000002": {
      "index": ".ds-my-data-stream-2023.04.19-000002",  <1>
      "managed_by_lifecycle": true,                           <2>
      "index_creation_date_millis": 1681919221417,
      "time_since_index_creation": "6.85s",             <3>
      "lifecycle": {
        "enabled": true,
        "data_retention": "30d"                         <4>
      }
    },
    ".ds-my-data-stream-2023.04.17-000001": {
      "index": ".ds-my-data-stream-2023.04.17-000001",  <5>
      "managed_by_lifecycle": true,                           <6>
      "index_creation_date_millis": 1681745209501,
      "time_since_index_creation": "48d",               <7>
      "rollover_date_millis": 1681919221419,
      "time_since_rollover": "6.84s",                   <8>
      "generation_time": "6.84s",                       <9>
      "lifecycle": {
        "enabled": true,
        "data_retention": "30d"                         <10>
      }
    }
  }
}
```

1. The name of the backing index.
1. This index is managed by a data stream lifecycle.
1. The time since this index was created.
1. The data retention for this index is at least 30 days, because the lifecycle was updated recently.
1. The name of the backing index.
1. This index is managed by the built-in data stream lifecycle.
1. The time since this index was created.
1. The time since this index was [rolled over](/manage-data/lifecycle/index-lifecycle-management/rollover.md).
1. The time used to determine when it's safe to delete this index and its data.
1. The data retention for this index is at least 30 days, because the lifecycle was updated recently.

:::
:::::

## Configure frozen {{search-snaps}} [configure-dlm-searchable-snapshots]
```{applies_to}
stack: ga 9.5+
serverless: unavailable
```

Add `frozen_after` to the lifecycle so older backing indices become partially mounted {{search-snaps}} on the frozen tier.
For how conversion works, operational behavior, and troubleshooting blocked conversions, refer to [](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md).

1. Confirm that a default snapshot repository is registered. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).
1. Set `frozen_after` (and optionally `data_retention`) on the data stream lifecycle.
   You can't use the **{{index-manage-app}}** tools in {{kib}} for this step.
   Use the [PUT data stream lifecycle API]({{es-apis}}operation/operation-indices-put-data-lifecycle):

    ```console
    PUT _data_stream/my-data-stream/_lifecycle
    {
      "data_retention": "365d",
      "frozen_after": "30d"
    }
    ```

    In this example, backing indices older than 30 days are converted to frozen {{search-snaps}}.
    {{es}} can still delete indices when their `generation_time` exceeds the effective retention period, even after they are frozen.

1. Verify the lifecycle configuration and conversion status with the [explain data lifecycle API]({{es-apis}}operation/operation-indices-explain-data-lifecycle):

    ```console
    GET .ds-my-data-stream-*/_lifecycle/explain
    ```

    If conversions are blocked, refer to [Troubleshooting](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md#dlm-frozen-transition-troubleshooting).

## Remove the lifecycle for a data stream [delete-lifecycle]

Remove the lifecycle with the **{{index-manage-app}}** tools in {{kib}} or the {{es}} [delete lifecycle API]({{es-apis}}operation/operation-indices-delete-data-lifecycle).

:::::{tab-set}
:group: kibana-api
:::{tab-item} {{kib}}
:sync: kibana

To remove a data stream's lifecycle:

1. Find **{{index-manage-app}}** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Data Streams** tab.
1. Search for the data stream.
1. Select the data stream to view its details.
1. In the flyout, select **Manage → Edit data retention**.
1. Turn off **Enable data retention** and save your changes. The maintenance operations from the lifecycle no longer apply to the data stream or its backing indices.

    Reopen the data stream flyout to confirm the change. **Effective data retention** shows **Disabled**.

  ::::{image} /manage-data/images/elasticsearch-reference-lifecycle-disabled.png
  :alt: Index lifecycle status is disabled
  :width: 500px
  ::::
:::
:::{tab-item} API
:sync: api

To remove a data stream's lifecycle:

```console
DELETE _data_stream/my-data-stream/_lifecycle
```

After this request, the maintenance operations from the lifecycle no longer apply to the data stream or its backing indices.

Use the [explain API]({{es-apis}}operation/operation-indices-explain-data-lifecycle) again to see that the indices are no longer managed.

```console
GET .ds-my-data-stream-*/_lifecycle/explain
```

```console-result
{
  "indices": {
    ".ds-my-data-stream-2023.04.19-000002": {
      "index": ".ds-my-data-stream-2023.04.19-000002",  <1>
      "managed_by_lifecycle": false                           <2>
    },
    ".ds-my-data-stream-2023.04.17-000001": {
      "index": ".ds-my-data-stream-2023.04.19-000001",  <3>
      "managed_by_lifecycle": false                           <4>
    }
  }
}
```

1. The name of the backing index.
1. The data stream lifecycle does not manage the index.
1. The name of another backing index.
1. The data stream lifecycle does not manage the index.
:::
:::::

## Manage data retention on the Streams page [data-retention-streams]
```{applies_to}
serverless: ga
stack: preview =9.1, ga 9.2+
```

The **Streams** page in {{kib}} is a centralized interface for common data management tasks, including how your data streams retain data.

1. Find **Streams** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Each stream maps directly to an {{es}} data stream. Select a stream to view its details.
1. Select the **Retention** tab to set how long your stream retains data, and to see ingestion and storage size.
1. Select **Edit data retention** and choose to retain your data indefinitely, for a custom period, or by following an existing {{ilm-init}} policy. You can also use the retention configuration from the index template by turning on **Inherit from index template**. If you use this option, don't specify a custom retention period or policy.

For more information about the retention configuration options, refer to [](/solutions/observability/streams/configure-retention.md).

## Next steps [update-data-stream-lifecycle-next-steps]

* To create a stream and set its lifecycle before data arrives, refer to [Create a data stream with a lifecycle](/manage-data/lifecycle/data-stream/tutorial-create-data-stream-with-lifecycle.md).
* To see how effective retention is calculated, refer to [Setting retention for {{es}} data streams](/manage-data/lifecycle/data-stream/tutorial-data-stream-retention.md).
* To move older backing indices to searchable snapshots, refer to [{{search-snaps-cap}} for data streams](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md).
* To move a stream from {{ilm-init}} to a data stream lifecycle, refer to [Migrating {{ilm-init}}-managed data streams to data stream lifecycle](/manage-data/lifecycle/data-stream/tutorial-migrate-ilm-managed-data-stream-to-data-stream-lifecycle.md).

## Related pages [update-data-stream-lifecycle-related]

* [Data stream lifecycle](/manage-data/lifecycle/data-stream.md)
* [Data streams](/manage-data/data-store/data-streams.md), including [backing indices](/manage-data/data-store/data-streams.md#backing-indices)
* [{{ilm-cap}}](/manage-data/lifecycle/index-lifecycle-management.md)
