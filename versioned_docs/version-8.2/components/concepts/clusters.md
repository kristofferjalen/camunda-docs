---
id: clusters
title: "Clusters"
description: "Learn more about the clusters available in your Camunda 8 plan."
---

A cluster is a provided group of nodes that run Camunda 8. By default, Camunda 8 clusters are production-ready.

Enterprise plan customers can create as many production or development clusters as they want. Starter plan customers are limited based on the [fair usage limits of the plan](https://camunda.com/legal/fair-usage-limits-for-starter-plan/).

Production clusters come in three sizes: small (S), medium (M), and large (L). To learn more about the size of cluster best suited for your use case, see our [Best Practices](/components/best-practices/best-practices-overview.md) for more information on [sizing your runtime environment](/components/best-practices/architecture/sizing-your-environment.md#sizing-your-runtime-environment).

The following table shows each plan and available type or size of cluster:

|              | Development | Production - S | Production - M | Production - L |
| ------------ | ----------- | -------------- | -------------- | -------------- |
| Free Trial   | \-          | X              | \-             | \-             |
| Free         | \-          | \-             | \-             | \-             |
| Professional | X           | X              | \-             | \-             |
| Enterprise   | X           | X              | X              | X              |

When you deploy and execute your [BPMN](/components/modeler/bpmn/bpmn.md) or [DMN](/components/modeler/dmn/dmn.md) models on a production cluster, this might impact your monthly (Professional) or annual (Enterprise) total fee, meaning the more you execute your models, the higher your total fee may be.

## Free Trial cluster

Free Trial clusters have the same functionality as a production cluster, but are size small and only available during your trial period. You cannot convert a Free Trial cluster to a different kind of cluster.

Once you sign up for a Free Trial, you are able to create one production cluster for the time of your trial.

When your Free Trial plan expires, you are automatically transferred to the Free plan. This plan allows you to model BPMN and DMN collaboratively, but does not support execution of your models. Any cluster created during your free trial is deleted, and you cannot create new clusters.

### Auto-pause

Free Trial `dev` (or untagged) clusters are automatically paused eight hours after a cluster is created or resumed from a paused state. Auto-pause occurs regardless of cluster usage.

You can resume a paused cluster at any time, which typically takes five to ten minutes to complete.

- Clusters tagged as `test`, `stage`, or `prod` do not auto-pause.
- Paused clusters are automatically deleted after 30 consecutive paused days. You can change the tag to avoid cluster deletion.
- No data is lost while a cluster is paused. All execution and configuration is saved, but cluster components such as Zeebe and Operate are temporarily disabled until you resume the cluster.

:::tip

To prevent auto-pause, you can:

- Tag the cluster as `test`, `stage`, or `prod` instead of `dev`.
- [Upgrade your Free Trial plan](https://camunda.com/pricing/) to a Starter, Professional, or Enterprise plan.

:::

## Development clusters

Development clusters are recommended for development, testing, proof of concepts, and demos.

Professional plan users have the option to create **development clusters**, offering free execution for development. Deployment and execution of models (process instances, decision instances, and task users) is provided at no cost.

Additionally, the following applies to **development clusters**:

- **Cluster is not high-available & less hardware**: Reduced hardware resources and availability compared to production cluster (for example, one Zeebe node only).
- **Shorter history of processes and decisions**: Data retention in Operate, Optimize, and Tasklist is reduced to one day. For example, pending or historical process instances are deleted after one day.