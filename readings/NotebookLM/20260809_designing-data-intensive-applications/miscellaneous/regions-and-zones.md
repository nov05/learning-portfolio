Page 236 / 673 
 

```text
Regions and Availability Zones

We use the term region to refer to one or more datacenters in a single geographic
location. Cloud providers locate multiple datacenters in the same geographic region.
Each datacenter is referred to as an availability zone or simply zone. Thus, a single
cloud region is made up of multiple zones. Each zone is a separate datacenter located
in separate physical facility with its own power, cooling, and so on.

Zones in the same region are connected by very high-speed network connections.
Latency is low enough that most distributed systems can run with nodes spread
across multiple zones in the same region as though they were in a single zone.
Multi-zone configurations allow distributed systems to survive zonal outages where
one zone goes offline, but they do not protect against regional outages where all
zones in a region are unavailable. To survive a regional outage, a distributed system
must be deployed across multiple regions, which can result in higher latencies, lower
throughput, and increased cloud networking bills. We will discuss these trade-offs
more in “Multi-leader replication topologies” on page 218. For now, just know that
when we say region, we mean a collection of zones/datacenters in a single geographic
location.
```