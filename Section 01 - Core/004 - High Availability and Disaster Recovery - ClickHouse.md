
##### Note:

- [High Availability and Disaster Recovery - ClickHouse](https://docs.fortinet.com/document/fortisiem/7.1.6/high-availability-and-disaster-recovery-procedures-clickhouse/933956/high-availability-and-disaster-recovery-clickhouse)

- we have only one supervisor node in any FortiSIEM deployment and this supervisor is considered as a single point of failure.
- so the high availability is here is focusing only on the supervisor because the workers and collector, they have native resiliency. for example,
	- in the worker case - you can have multiple workers and multiple shards
	- in the collector case - you can put the load balancer in front of the collectors and design your resiliency based on N + 1
		- if you need to survive one lost collector, your sizing will be revealing only 5 collectors, you can put 6 collectors. so design 6 collector or put 6 collector behind the load balancer. thanks to supervisor round-robin mechanism

- in FortiSIEM we can have multiple supervisor nodes and they can run in active mode. to achieve this you can install multiple supervisor nodes and only one node is acting as a leader while the others will be acting as followers

![[FortiSIEM U/Section 01 - Core/004.png]]

- note that the workers are aware about the leader. each worker has a communication with all the supervisors, so the worker is aware about the leader and interact with one leader at a time. 
- if the supervisor leader fails, some other supervisors can take over. however, taking over is not that straightforward and not done automatically unless until you actually access one of the followers and activate some script to promote this follower as a new leader.
	- one more limitation - you actually have the grace period of two weeks for the new leader to activate the license, because the license is bound to the `hardware ID` of the node itself.

![[FortiSIEM U/Section 01 - Core/005.png]]

- good thing is FortiSIEM provide you with 2 weeks of grace period in case you are able to recover the original leader, so don't need to activate the new license.
- third limitation - need to have an external load balancer (e.g., F5 Big-IP or fortinet fortiweb appliances)
	- load balancer is an optional thing, if you don't have the load balancer, the collectors and agents are directly connected.
	- only limitation is you need to change the DNS or repoint your DNS to the new follower and also for the agents and the collectors


- So basically, to configure the `high availability`, you need to install the license for the HA, this is the prerequisite to be able to see the other supervisors and then you have to go to `Admin > License > Nodes` and add the supervisors for being a primary follower 


- In failure scenario, in case the whole site is down as indicated by red box, all components will start talking to the secondary site using DNS redirection.
	- it also requires that you access the secondary site and promote the supervisor to become the primary node. (See the documentation to do this - need to run script)

![[FortiSIEM U/Section 01 - Core/006.png]]

- After the failure, you will promote the secondary supervisor to become the primary. and if the primary site back to operation, leave it as secondary for some time until it sync all the lost data and you have option after that to promote it as primary.

*Note:*
- Keep the same number of `shards`, so keep the same number of workers in each shard in the primary and secondary site. so if you have two workers in shard 1 in site 1 (primayr) you need to install another two workers in shard 1 in site 2 (secondary).
	- the specs (CPU, memory) should be identical
- this will ensure that you have HA (High Availability) in the primary site and DR (Disaster Recovery) in case of failure. but, there is no HA for secondary site.
	- you just need to access supervisor from secondary site and promote it from secondary to become primary.
	- at this time, when you make this, the two sites will become independent from each other, because the primary site is assumed down as soon as you enable/activate the script for promoting this from secondary to primary

- check with documentation for configuration of `external load balancer configuration` for FortiWeb load balancer
	- for load balancing algorithm, there are two options
		- `least connection`
		- `round robin`