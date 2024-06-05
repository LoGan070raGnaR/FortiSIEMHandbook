- just follow the same installation process of FortiSIEM
- only two things to remember:
	- first thing - layout of the machine will just require two partition instead of five
	- second thing - FortiCollector doesn't require the license, all you need to do is connect it to or register it with the supervisor, that's all, you will find the services coming up after this step

- while installing, in the `Configuration Target`, don't choose the `Supervisor` instead choose `Collector`
- after installation login and check the status of the service using `phstatus` command. it will show FortiSIEM collectors as `DOWN`, because we haven't registered the collector yet with the FortiSIEM.
	- all we need to do is go to the `FortiSIEM`, then `ADMIN > Collector > New (Name, Guranteed EPS, optionally Event workers for medium to large environment)
- now we need to issue a command to register collector with the supervisor in PuTTY, use IP of collector
- Syntax:
```bash
/opt/phoenix/bin/phProvisionCollector -add <user> '<password>' <Super IP or Host> <Organization> ORG_Name Collector
```
- E.g.,
```bash
/opt/phoenix/bin/phProvisionCollector -add admin '<password>' <Super IP or Host> Super DC_Collector
```

*Note:*
- for enterprise version we don't need `Organization` ID, for multi-tenant version we need it.
- after successful installation it will show 'The collector is successfully registered' message. then it will reboot and the services should start and the collector will connect with supervisor
	- To check this, go to `FortiSIEM > ADMIN > Device Support > Health < Collector Health`


