#  Setup Native Scheduling ElasticSearch Cluster Locally + AzureRM Using Nomad (Part 1)

## TL;DR
In order to run distributed worklaod natively; there is no need to run it in Docker.  Show how it is done using Nomad to run a native ElasticSearch single node and 

### Getting Started

For Part 1; we will run Consul + Nomad in local development mode.  This will be used to deploy the workloads.

Optionally, you can use the excellent community contributed hashi-ui to track the services.

Download the latest ElasticSearch binaries (ES-5.5) to be served locally and not need to continuously download it again.

### Single Node

In order to get started, get the latest binaries 


It ccan also be downloaded direct from ElasticSearch but to be faster we store and serve it locally.
```hcl
      artifact {
        source = "http://localhost:8080/elasticsearch-5.5.0.zip"
        // source = "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.0.zip"
        destination = "local"
        options {
          checksum = "sha1:b5835f207cec4ed73758f5f1640ede59851f873f"
        }
      }
```

The environment variable that needs to be passed in
```hcl
      env {
        ES_CLUSTER_NAME = "docker-cluster"
        ES_HOME = "local/elasticsearch-5.5.0"
      }
```

For it to be run using the Java driver; the startup script needs to be ported to be

Where the full Single Node ElasticSearch Job Specification

```
<script src="https://gist.github.com/leowmjw/5ffd63b70fb56565f5269b398ac3cacf.js"></script>
```

### Cluster Node

The relevant portion of the ElasticSearch config is as per below.  There is a hack in there if IPv6 localhost is detected, the Address used should be localhost as there seems to be some problem with the local Consul run in dev mode .. :(

```bash
...
          cluster.name: {{ env "ES_CLUSTER_NAME" }}
          network.host: {{ env "attr.unique.network.ip-address" }}
          discovery.zen.minimum_master_nodes: 2
          # network.publish_host: {{ env "attr.unique.network.ip-address" }}
          {{ if service "escluster-transport"}}discovery.zen.ping.unicast.hosts:{{ range service "escluster-transport" }}
            - {{ if eq .Address "::1" }}localhost{{ else }}{{ .Address }}{{ end }}:{{ .Port }}{{ end }}{{ end }}
          http.port: {{ env "NOMAD_HOST_PORT_eshttp" }}
          transport.tcp.port: {{ env "NOMAD_HOST_PORT_estransport" }}
...
```

The important part is the need the dynamic host ports to be filled in that is indicated earlier under the service stanza pointing to the ES Transport port.

```bash
      service {
        name = "escluster-transport"
        port = "estransport"
      }
```

### ESCluster UI Cerebro
The deplioyment of cluster ElasticSearch can be viewed using the nicely done UI called erebro

Showing the flexibility

### What About Docker?
It is not a mutually exclusive decision, the beauty of Nomad is that it can run different type of workloads together; which also includes Docker.  So, for example, we can run the ESCluster UI Cerebro as   

### Output
You can confirm the ES by doing the foloowing curl 
This is how to index data ..
Also, the output for 

### What Next?

In the nexxt article, we'll use the latest Nomad Box ; and explore more intricaies of the different ElasticSearch node types; and how it can be made more high availability
