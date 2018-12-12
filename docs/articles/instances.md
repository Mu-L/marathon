# Instances

## What is an Instance?
Instances are the Marathon representation of [⇒Tasks](tasks.md) or [Task Groups](task-groups.md) in Mesos. An instance can exist even if there is currently no Task associated. We introduced this abstraction in order to support Mesos TaskGroups (called [⇒Pods](pods.md) in Marathon), where an instance can contain more than one Task. Beyond support for Pods, this abstraction also allows us to represent scheduled instances, which do not yet have Mesos Tasks associated with them. 

### Instance Goals
Depending on the goal of a given instance, Marathon will take actions to reach that goal: it might match offers in order to launch a Task or TaskGroup. In this case, these Tasks will eventually be associated with the instance. If an instance shall be stopped, Marathon will send kill requests to Mesos and eventually expunge the instance from its state.

Currently, instance goals are only used internally, but are reflected in the API. Supported goals are
* `Running` means the goal is to have a Mesos Task or TaskGroup associated with this instance, which should be running all the time.
* `Stopped` means, all Mesos Tasks or TaskGroups associated with this instance shall be stopped. The instance and all its metadata will be retained. It is possible to change the goal to `Running` again.
* `Decommissioned` means the Mesos Task or TaskGroup associated with this instance will be killed, and reservations and volumes will be destroyed. The instance cannot be revived, and once in a reportedly terminal state, it will be expunged from Marathon's state.

See [⇒Instance Life Cycle](instance-lifecycle.md) for more information on instance life cycle and goals.

### Instances in the API
The `v2/apps` and `v2/tasks` API endpoints do not provide any means of direct interaction with instances. For historical reasons, these endpoints list tasks, and only allow to interact with them. The `v2/pods` endpoint on the other hand was build with instances as an abstraction for Mesos tasks, and thus does only allow direct interaction with instances, not with tasks.

### Instance Ids
The relation between instances and their Tasks is reflected in their names. An app with name `/my-service` will result in instances named `my-service.<UUID>`. The first Task associated with this instance will be named `my-service.instance-a41beca1-fa49-11e8-93c4-269daf75ef3f.$anon.1` where `$anon` designates the container as an anonymous container. The `.1` reveals the Task is the first task incarnation of this instance. Accordingly, `my-service.instance-<UUID>.$anon.42` will tell you that this is task incarnation number 42. This means 41 previous tasks for this instance became terminal, either because they exited normally or abnormally, or were killed by Marathon during a scale or upgrade operation.

A pod instance will have named containers that inherit the names from the respective PodDefinition, e.g. `my-service.instance-395c02eb-25c5-4a3b-8a42-3d2635264058.rails.1` and `my-service.instance-395c02eb-25c5-4a3b-8a42-3d2635264058.sidecar.1`.

## How can I use Instances?
Instances are created based on [⇒Service](services.md) specifications which can be either an [⇒AppDefinition](applications.md) or a [⇒PodDefinition](pods.md). Instances are implicitly created according to their App or Pod definition, which contains an instances property that designates the required amount of instances.

## Which related concepts should I understand?
When upgrading a Service during a [⇒Deployment](deployments.md) from one version to another, Marathon can create more instances than specified in either Service definition. It will do so if allowed via the [⇒upgradeStrategy](upgrade-strategy.md) configuration.
When an instance is running on an agent disconnected from the cluster, it will eventually be reported as unreachable. According to the Service’s [⇒unreachableStrategy](unreachable-strategy.md), Marathon can schedule a replacement, even though the unreachable instance might still be running. When an unreachable instance becomes becomes reachable again, Marathon will scale down accordingly to the required amount of instances. It will decide over which instance shall be killed based on the [⇒killSelection](kill-selection.md).

## Are there any limitations or things to consider?
_(This section could point out known limitations wrt number of instances.)_

## Links
* [⇒Scaling Applications](../tutorials/apps-scaling.md)
* [⇒Scaling Pods](../tutorials/pods-scaling.md)
* [⇒Killing Instances](../tutorials/instance-operations.md#delete-instances)

## Examples
* [⇒Simple Application](examples/app-simple.json) definition configuration 3 instances
* [⇒JSON Representation](examples/app-representation.json) of an instance via v2/tasks