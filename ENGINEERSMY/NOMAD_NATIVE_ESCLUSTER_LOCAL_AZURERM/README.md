#  Setup Native Scheduling ElasticSearch Cluster Locally + AzureRM Using Nomad (Part 1)

## Objective
- Investigate how a Java-based application can be ported to run at scale on [Nomad](https://www.nomadproject.io); highlighting potential gotchas and pitfalls when performing the port.  Also show how a native Java workload can easily be run together with Docker workload using Nomad.

## Audience
- Developers wanting to setup and use ElasticSearch easily but still be able to test in production-like setup
- Operators getting started with running a High Availability ElasticSearch Cluster in Nomad

## References
- Nomad Job Specification: https://www.nomadproject.io/docs/job-specification/index.html
- Native Java Workload Example, ElasticSearch: [Nomad Samples](https://www.github.com/leowmjw/nomad-samples) 
- Example job specification is under the folder: `/java/go-elasticsearch`.  Named `elasticsearch.nomad` and `cluster-elasticsearch.nomad`.  A Docker example is also provided `docker-cluster-elasticsearch.nomad`.
 
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
- [What About Docker?](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#what-about-docker-)
- [ESCluster UI Cerebro](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#escluster-ui-cerebro)
- [Output](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md.md#output)
- [What Next?](./Nomad-Native-Scheduling-ESCluster-Local-AzureRM.md#what-next-)
