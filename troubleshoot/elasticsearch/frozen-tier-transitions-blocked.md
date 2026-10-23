---
navigation_title: Frozen tier transitions
description: Diagnose and resolve data stream backing indices that are not converting to frozen {{search-snaps}}.
type: troubleshooting
applies_to:
  stack: ga 9.5
  serverless: unavailable
products:
  - id: elasticsearch
---

# Backing indices are not moving to the frozen tier [frozen-tier-transitions-blocked]

Older backing indices on a data stream with `frozen_after` stay on hot or warm nodes instead of converting to [frozen {{search-snaps}}](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md). Until the conversion finishes, those indices keep using local storage instead of the frozen tier.

## Symptoms

- The [explain data lifecycle API]({{es-apis}}operation/operation-indices-explain-data-lifecycle) reports a conversion error on a backing index.
- {applies_to}`stack: ga 9.6` The `dlm_frozen_transitions` indicator of the [health report API]({{es-apis}}operation/operation-health-report) is `yellow` or `unknown`.
- Backing indices older than `frozen_after` remain on non-frozen tiers.

## Diagnosis

Inspect the backing index with the [explain data lifecycle API]({{es-apis}}operation/operation-indices-explain-data-lifecycle):

```console
GET .ds-my-stream-*/_lifecycle/explain
```

Note the error on each blocked index. For transient failures, {{dlm-init}} records the error and retries the conversion on the next poll cycle. Each conversion step is idempotent, so retries resume from the point at which the error occurred. Conversions stop retrying when the explain output reports an unrecoverable error, such as a missing snapshot repository or a license compliance failure.

### Check cluster-wide frozen transition health
```{applies_to}
stack: ga 9.6
```

Use the health report to see overdue frozen conversions across the cluster:

```console
GET _health_report/dlm_frozen_transitions
```

Read `status`, `symptom`, and each `diagnosis.action`. In `details`, check `transitions_enabled`, `default_repository_configured`, and `overdue_indices_count_by_state` (`unmarked`, `marked`, or `queued`).

A conversion that is already running is not overdue, so the indicator stays `green` while that work is in progress.

- `green`: No overdue conversions, or every overdue conversion is already running.
- `yellow`: Conversions are blocked, queued, or waiting while frozen transitions are disabled.
- `unknown`: The master node has not published health information for three [`dlm.frozen_transitions.health.publish_interval`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#dlm-frozen-transition-health-publish-interval) intervals.

An index is overdue only after it has been past `frozen_after` for longer than [`dlm.frozen_transitions.health.stuck_threshold`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#dlm-frozen-transition-health-stuck-threshold).

## Resolution

Use the option that matches the error or diagnosis.

{applies_to}`stack: ga 9.6` When the health report is `yellow`, follow `diagnosis.action`.

1. If the error or diagnosis reports a missing default snapshot repository, register one. Refer to [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md).
2. If the error reports a license compliance failure, renew the license. Refer to [{{es}} pricing and subscriptions]({{subscriptions}}).
3. If frozen transitions are disabled, set [`dlm.frozen_transitions.enabled`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#dlm-frozen-transition-enabled) to `true`. Transitions that are already running continue either way. Indices that are marked or queued stay overdue until you re-enable the setting:

    ```console
    PUT _cluster/settings
    {
      "persistent": {
        "dlm.frozen_transitions.enabled": true
      }
    }
    ```

4. If the diagnosis reports queued transitions, inspect the `dlm_frozen_transition` thread pool with the [cat thread pool API]({{es-apis}}operation/operation-cat-thread-pool):

    ```console
    GET _cat/thread_pool/dlm_frozen_transition?v
    ```

    A persistently full queue means conversions complete more slowly than indices become eligible. Wait for in-flight work to drain, or add capacity so more conversions can run at once.

5. If the diagnosis reports that the frozen transition service is not running on the current master node, check that node's logs for frozen transition errors. Restart the master node if the service stays down.
6. {applies_to}`stack: ga 9.6` If the health indicator is `unknown`, check the current master node's logs and confirm the master is publishing frozen transition health on the [`dlm.frozen_transitions.health.publish_interval`](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#dlm-frozen-transition-health-publish-interval) schedule.

## Resources

- [{{search-snaps-cap}} for data streams](/manage-data/lifecycle/data-stream/dlm-searchable-snapshots.md)
- [Data stream lifecycle](/manage-data/lifecycle/data-stream.md)
- [Manage snapshot repositories](/deploy-manage/tools/snapshot-and-restore/manage-snapshot-repositories.md)
- [Frozen tier transition settings](elasticsearch://reference/elasticsearch/configuration-reference/data-stream-lifecycle-settings.md#_frozen_tier_transition_settings)

:::{tip}
If you have an [Elastic subscription](https://www.elastic.co/pricing), then you can [contact Elastic support](/troubleshoot/index.md#contact-us) for assistance.
:::
