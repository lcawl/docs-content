# Frozen shards decider [autoscaling-frozen-shards-decider]

The [autoscaling](../../../deploy-manage/autoscaling.md) frozen shards decider (`frozen_shards`) calculates the memory required to search the current set of partially mounted indices in the frozen tier. Based on a required memory amount per shard, it calculates the necessary memory in the frozen tier.

## Configuration settings [autoscaling-frozen-shards-decider-settings]

`memory_per_shard`
:   (Optional, [byte value](elasticsearch://reference/elasticsearch/rest-apis/api-conventions.md#byte-units)) The memory needed per shard, in bytes. Defaults to 2000 shards per 64 GB node (roughly 32 MB per shard). Notice that this is total memory, not heap, assuming that the Elasticsearch default heap sizing mechanism is used and that nodes are not bigger than 64 GB.


