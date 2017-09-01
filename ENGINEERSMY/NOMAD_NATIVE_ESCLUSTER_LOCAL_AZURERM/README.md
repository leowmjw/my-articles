#  Setup Native Scheduling ElasticSearch Cluster Locally + AzureRM Using Nomad

## Objective
- Investigate how a Java-based application can be ported to run at scale on [Nomad](https://www.nomadproject.io); seeing the gotchas and .  quick comparison against running in Docker instead

## Audience
- Developers wanting to setup and use ES easily but still getting assurance doing the right thing
- Operators getting started with running a HA ElasticSearch Cluster

## References
- Java example - ESClsuter - Nomad-Samples [Nomad Samples](https://www.github.com/leowmjw/nomad-samples) code
- Script is under the folder `/lxd/scripts/initlxd.sh`.
                              
- Docker Elasticserch
 
## Highlights 
- How to port over bash startup script into Nomad Job Specs
- Simple single node
- Complex multiple node
- Multiple environments at once ..
- ESCLuster GUI using Cerebro
- JVM Gotchas
- Consul template Gotchas
- Categories of ElasticSearch nodes: master, data, injestors, confirmer??
- Compare against Docker run ..
- What next; running ES in production, failure modes, monitoring, data loss + recovery ..

## Content link
- [Getting Started](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#getting-started)
- [Single Node](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#single-node)
- [Cluster Node](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#cluster-node)
- [ESCluster UI Cerebro](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#escluster-ui-cerebro)
- [What About Docker?](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#what-about-docker-)
- [Output](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md.md#output)
- [What Next?](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#what-next-)
