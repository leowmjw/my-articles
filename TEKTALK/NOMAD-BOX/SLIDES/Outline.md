# OUtline + NOtes

An opinionated 

## Nomad Box - Fabric to Deploy services
- Supported Service Type / containers
- Supported 
- Automatic self-healing + self-routing
- Fancy Dashboard (all your resources!)

## Detailed  Breakdown of Components of Nomad Box

- Brief explaination of Consul?
- Brief explaination of Nomad?
- Brief explaination of Traefik?

### Foundation Nodes
- Manages the cluster; holds the Bastion Host 
- sshuttle is great "poor-man"'s VPN tunnel (does not work well for UDP tho)
- Runs Consul Servers
- Runs Nomad Servers

### Director Nodes
- Has Public IP
- L7 load balance using Traefik
- Runs Consul Agent Clients
- Run Nomad Agent Clients
- Lighter load labels

### Worker Nodes
- Only Private IP
-
- Can run Windows workload
- Put storage layer here?
- Different labels; tags can be used for selection

### Demo 
- Pray to the Demo ${DEITY} :P
- Starting up the Cluster --> https://asciinema.org/a/791pz66bwzz11x729j1z5cngf
- Check your Azure Portal to see it completed ..
- For now only the stateless parts
- Run the whoami load; show the load balance in Traefik
-

## Using Nomad Box in Development, Staging, Production (not yet..)
-
- Automap using Caddy to work as a Reverse Proxy
- Can be done for multiple enviroments
- For laptop; uses mDNS (in the *.local domain)

## Technology Agnostic
- Use the best features of individual cloud/native
- Idempotent
- Machine leverage to get the best (bin-packing), costing ..
- Not tied to any specific technology/services
- Carve out the interface; use as per implementation

## Distributed SYstems
- Consul can 
- Also serves can configuration toggle/ feature toggle
- Discovered by SDK or via DNS (show demo)

### Service Discovery

### Self Seevicing
- Valut can audit
- 

### Scheduling / Orchestration
- Nomad can deploy the following "containers" to be executed
- Feedback loop 

### Load  Balancing / Error Handling
- Needs 
- Can still put something like Azure Traffic Manager in front
- Should be as transparent as possible to the developers (cross custting)
- Linkerd for internal mesh, error handling, 
- Blue / Green deployment

### Autopilot Pattern
- Enables migrating even legacy applications not designed for Docker (e.g Wordpress)
- Not splitting responsibility of liveliness and availability
- Can attach all the needed co-pilot processes; logging, helpers
- Helps "Stateful" services work in a stateless environment

## Alternatives
- Docker Swarm - 
- Kubernetes - the current "Hot" service; looks complex

## Closing / What audience should Learn
- What can make up a simple Cluster Fabric for running microservices
- How and what pieces
- A method for "simulating" locally
- Applications should be in charge of their own Stateful

## Future Ideas /  demo
- Better ACL, Vault integration
- Better internal mesh traffic via linkerd
- Windows workload (Music Store example) + AzureDB
- "Stateful"-services via Autopilot Pattern
- ArangoDB, Expensify's BedrockDB
- PerconaDB MySQL + PostgreSQL
- Fully automated Windows Nano Server deployed via Terraform

## Resources
- How Raft Works - <http://thesecretlivesofdata.com/raft/>
- http://thesecretlivesofdata.com/raft/

