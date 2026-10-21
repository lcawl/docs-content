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

Change how long an existing {{es}} data stream keeps its data or remove that lifecycle.
This tutorial is for you if the data stream already exists and you want to update it, not the index template that created it.

By the end of this tutorial, you can:

* Set or change how long a data stream keeps data.
* {applies_to}`stack: ga 9.5+` {applies_to}`serverless: unavailable` Move older [backing indices](/manage-data/data-store/data-streams.md#backing-indices) to the frozen tier.
* Remove the data stream lifecycle.

:::{note}
These steps are for data stream lifecycle only.
For {{ilm-init}}, refer to [](/manage-data/lifecycle/index-lifecycle-management.md).
For a comparison of these features, go to [](/manage-data/lifecycle.md).
:::

## Before you begin [update-data-stream-lifecycle-prereqs]

To update the data stream in this tutorial, you need:

* An existing data stream. To create one, refer to [Create a data stream with a lifecycle](/manage-data/lifecycle/data-stream/tutorial-create-data-stream-with-lifecycle.md).
* Access to [{{dev-tools-app}} Console](/explore-analyze/query-filter/tools/console.md) in {{kib}}, or another way to send {{es}} API requests.
* These [index privileges](elasticsearch://reference/elasticsearch/security-privileges.md#privileges-list-indices) on the data stream and its backing indices.

  * `manage_data_stream_lifecycle`, to update or remove the lifecycle. This privilege also covers the explain API.
  * `view_index_metadata`, if you only read lifecycle state with the explain API.

* {applies_to}`stack: ga 9.5+` {applies_to}`serverless: unavailable` A default snapshot repository, if you move older backing indices to the frozen tier. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).

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
1. In the data stream details pane, adjust the data retention settings:

   * {applies_to}`stack: ga 9.5+` Click the actions icon {icon}`gear` and select **Edit data lifecycle**.
     You can do any of the following:

     * On the **Successful data** tab, select the **Data stream lifecycle** method.

       Select the **Delete phase** to choose how long to retain your data, in days, hours, minutes, or seconds. If you deselect the delete phase, {{es}} does not delete the data.

       Select the **Frozen phase** to choose when to move your data to [{{search-snaps}}](#configure-dlm-searchable-snapshots), in days, hours, hours, minutes, or seconds.
     * On the **Failed data** tab, select **Enable failure store** to turn on a [failure store](/manage-data/data-store/data-streams/failure-store.md). Select how long to retain failure store data, in days, hours, minutes, or seconds.

     Click **Apply** to save the changes.
  
   * {applies_to}`stack: ga 9.0-9.4` Select **Manage** → **Edit data retention**.
     You can do any of the following:

     * Select how long to retain your data, in days, hours, minutes, or seconds.
     * Select **Keep data indefinitely**. The data stream stays managed but {{es}} does not delete the data. Managing a {{tsds}}, such as logs or metrics, helps {{es}} store your data even when you don't set a retention period.
     * Turn off **Enable data retention** to stop data stream lifecycle management for this data stream.

1. Reopen the data stream flyout to confirm the change. Lifecycle changes apply to every backing index of the data stream.

   * {applies_to}`stack: ga 9.0-9.4` The flyout shows the effective data retention settings.
   * {applies_to}`stack: ga 9.5+` The flyout shows the successful and failure ingest lifecycle details.

$$$data-retention-streams$$$
Alternatively, you can use the **Streams** page in {{kib}}, which is a centralized interface for common data management tasks, including how your data streams retain data. Refer to [](/solutions/observability/streams/configure-retention.md).
:::
:::{tab-item} API
:sync: api

To change the data retention settings for a data stream:

* Set an indefinite retention period so {{es}} does not delete the data. For example:

    ```console
    PUT _data_stream/my-data-stream-test/_lifecycle
    { } <1>
    ```

    1. An empty payload means that your data stream is managed but {{es}} does not delete the data. Managing a {{tsds}}, such as logs or metrics, helps {{es}} store your data even when you don't set a retention period.

* Or set a retention period. For example:

    ```console
    PUT _data_stream/my-data-stream-test/_lifecycle
    {
      "data_retention": "30d" <1>
    }
    ```

    1. The retention period of this data stream is 30 days. {{es}} can delete data older than 30 days.

The lifecycle change applies to every backing index of the data stream.
To check the settings, use the {{es}} [explain API]({{es-apis}}operation/operation-indices-explain-data-lifecycle):

```console
GET my-data-stream-test/_lifecycle/explain
```

**Result:**

```console-result
{
  "indices": {
    ".ds-my-data-stream-test-2026.10.17-000001": {
      "index": ".ds-my-data-stream-test-2026.10.17-000001", <1>
      "managed_by_lifecycle": true, <2>
      "index_creation_date_millis": 1792195511314,
      "time_since_index_creation": "2.97d", <3>
      "rollover_date_millis": 1792440980120,
      "time_since_rollover": "3.12h", <4>
      "generation_time": "3.12h", <5>
      "lifecycle": {
        "enabled": true,
        "data_retention": "30d", <6>
        "retention_determined_by": "data_stream_configuration"
      }
    },
    ".ds-my-data-stream-test-2026.10.19-000002": {
      "index": ".ds-my-data-stream-test-2026.10.19-000002",
      "managed_by_lifecycle": true,
      "index_creation_date_millis": 1792440980129,
      "time_since_index_creation": "3.12h",
      "lifecycle": {
        "enabled": true,
        "data_retention": "30d"
        "retention_determined_by": "data_stream_configuration"
      }
    }
  }
}
```

1. The name of the backing index.
1. This index is managed by a data stream lifecycle.
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

Configure the data stream lifecycle frozen phase with the **{{index-manage-app}}** tools in {{kib}} or the {{es}} [lifecycle API]({{es-apis}}operation/operation-indices-put-data-lifecycle).

For how conversion works, operational behavior, and troubleshooting blocked conversions, refer to [](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md).

:::::{tab-set}
:group: kibana-api
:::{tab-item} {{kib}}
:sync: kibana

To change the frozen phase settings for a data stream:

1. Confirm that a default snapshot repository is registered.
   To change the default snapshot repository, refer to [{{ech}}](/deploy-manage/tools/snapshot-and-restore/elastic-cloud-hosted.md#snapshot-repo-default) or [self-managed](https://www.elastic.co/docs/deploy-manage/tools/snapshot-and-restore/self-managed#snapshot-repo-default) instructions.
1. Find **{{index-manage-app}}** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Data Streams** tab.
1. Search for the data stream.
1. Select the data stream to view its details.
1. In the data stream details pane, click the actions icon {icon}`gear` and select **Edit data lifecycle**.
1. On the **Successful data** tab, select the **Data stream lifecycle** method.
1. Select the **Frozen phase** to choose when to move your data to {{search-snaps}}, in days, hours, hours, minutes, or seconds.
This period of time must be smaller than the period specified for the delete phase.
1. Click **Apply** to save the changes.

:::
:::{tab-item} API

Add `frozen_after` to the lifecycle so older backing indices become partially mounted {{search-snaps}} on the frozen tier.

1. Confirm that a default snapshot repository is registered. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).
1. Set `frozen_after` (and optionally `data_retention`) on the data stream lifecycle.
   Use the [PUT data stream lifecycle API]({{es-apis}}operation/operation-indices-put-data-lifecycle):

    ```console
    PUT _data_stream/my-data-stream-test/_lifecycle
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
:::
:::::

## Remove the lifecycle for a data stream [delete-lifecycle]

Remove the lifecycle with the **{{index-manage-app}}** tools in {{kib}} or the {{es}} [delete lifecycle API]({{es-apis}}operation/operation-indices-delete-data-lifecycle).

:::::{tab-set}
:group: kibana-api
:::{tab-item} {{kib}}
:sync: kibana

<!-- TBD: Not applicable to Serverless per https://github.com/elastic/docs-content/pull/8021#discussion_r4272648315 ? -->

To remove a data stream's lifecycle:

1. Find **{{index-manage-app}}** in the navigation menu or use the [global search field](/explore-analyze/find-and-organize/find-apps-and-objects.md).
1. Select the **Data Streams** tab.
1. Search for the data stream.
1. Select the data stream to view its details.
1. In the data stream details pane, adjust the lifecycle settings:

   * {applies_to}`stack: ga 9.5+` Click the actions icon {icon}`gear` and select **Edit data lifecycle**. On the **Successful data** tab, select the **Data stream lifecycle** method and deselect all optional phases. Click **Apply** to save the changes.
   * {applies_to}`stack: ga 9.0-9.4` Select **Manage** → **Edit data retention**. Turn off **Enable data retention** and save your changes.
1. Reopen the data stream flyout to confirm the change.

   * {applies_to}`stack: ga 9.5+` **Successful ingest lifecycle** shows an infinity icon {icon}`infinity` to indicate that it keeps data indefinitely.
   * {applies_to}`stack: ga 9.0-9.4` **Effective data retention** shows **Disabled**.

<!-- NOTE: In 9.6 test these Kibana methods seem to have the same result as the API i.e. do make the stream `"managed_by_lifecycle": false` -->

:::
:::{tab-item} API
:sync: api

To remove a data stream's lifecycle:

```console
DELETE _data_stream/my-data-stream-test/_lifecycle
```

After this request, the maintenance operations from the lifecycle no longer apply to the data stream or its backing indices.

Use the [explain API]({{es-apis}}operation/operation-indices-explain-data-lifecycle) to verify that the indices are no longer managed.

```console
GET my-data-stream-test/_lifecycle/explain
```

```console-result
{
  "indices": {
    ".ds-my-data-stream-test-2026.10.17-000001": {
      "index": ".ds-my-data-stream-test-2026.10.17-000001", <1>
      "managed_by_lifecycle": false, <2>
      "unmanaged_reason": "Index [.ds-my-data-stream-test-2026.10.17-000001] belongs to data stream [my-data-stream-test] which does not have data stream lifecycle configuration." <3>
    },
    ".ds-my-data-stream-test-2026.10.19-000002": {
      "index": ".ds-my-data-stream-test-2026.10.19-000002",
      "managed_by_lifecycle": false,
      "unmanaged_reason": "Index [.ds-my-data-stream-test-2026.10.19-000002] belongs to data stream [my-data-stream-test] which does not have data stream lifecycle configuration."
    }
  }
}
```

1. The name of the backing index.
1. The data stream lifecycle does not manage the index.
1. {applies_to}`serverless: ga` {applies_to}`stack: ga 9.6+` The reason the data stream is not managed.
:::
:::::

## Next steps [update-data-stream-lifecycle-next-steps]

* To create a stream and set its lifecycle before data arrives, refer to [Create a data stream with a lifecycle](/manage-data/lifecycle/data-stream/tutorial-create-data-stream-with-lifecycle.md).
* To see how effective retention is calculated, refer to [Setting retention for {{es}} data streams](/manage-data/lifecycle/data-stream/tutorial-data-stream-retention.md).
* To move a stream from {{ilm-init}} to a data stream lifecycle, refer to [Migrating {{ilm-init}}-managed data streams to data stream lifecycle](/manage-data/lifecycle/data-stream/tutorial-migrate-ilm-managed-data-stream-to-data-stream-lifecycle.md).

## Related pages [update-data-stream-lifecycle-related]

* [](/manage-data/lifecycle/data-stream.md)
* [](/manage-data/data-store/data-streams.md)
* [](/manage-data/lifecycle/index-lifecycle-management.md)
* [](/deploy-manage/users-roles/cluster-or-deployment-auth/granting-privileges-for-data-streams-aliases.md)
* [](/manage-data/data-store/data-streams/manage-data-stream.md)