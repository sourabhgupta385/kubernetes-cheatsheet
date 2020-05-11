# Difference between ReplicationController & ReplicaSet

Replica Set is the next generation of Replication Controller. Replication controller is kinda imperative, but replica sets try to be as declarative as possible.

Replica Set                                          | Replication Controller
---------------------------------------------------- | ---------------------------------------------------
Replica Set supports the new set-based selector      | Replication Controller only supports equality-based selector
For eg: environment in (production, qa) This selects all resources with key equal to environment and value equal to production or qa                      | For eg: environment = production This selects all resources with key equal to environment and value equal to production
rollout command is used for updating the replica set | rolling-update command is used for updating the replication controller
Even though replica set can be used independently, it is best used along with deployments which makes them declarative                                    | This replaces the specified replication controller with a new replication controller by updating one pod at a time to use the new PodTemplate 