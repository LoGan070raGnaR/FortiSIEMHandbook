- here we will discuss how to automatically discover your devices, your applications, your users in your IT infrastructure, and build CMDB database based on that discovery

- for discovery there are multiple methods or protocols. not all the protocols/methods are applicable to all the devices
	- you can find the details about each and every device based on external device configuration documentation

- in `Dashboard > Identiy & Location`, you can see/discover the assets of devices and their location

![[FortiSIEM U/Section 02 - Device Discovery/001.png]]

##### FortiGate Discovery

```text
Discovering Devices
-------------------
FortiSIEM automatically discovers devices, applications, and users in your IT infrastructure and start monitoring them. You can initiate device discovery by providing the credentials that are needed to access the infrastructure component, and from there FortiSIEM will discover information about your component such as the host name, operating system, hardware information such as CPU and memory, software information such as running processes and services, and configuration information. Once discovered, FortiSIEM will also begin monitoring your component on an ongoing basis.

Protocol: SNMP
--------------
Used for: Availability and Performance Monitoring
-------------------------------------------------
Information Discovered: Host name, Hardware model, Network interfaces, Operating system version
-----------------------------------------------------------------------------------------------
Metrics collected: Uptime, CPU and Memory utilization, Network Interface metrics
================================================================================

1/ Setup in FortiGate
---------------------
config system interface
edit "port1"
 show
 set allowaccess ping https ssh http fgfm snmp
end
config system snmp sysinfo
 set status enable
 set description "FortiSIEM collector polling inbound on interface port 1"
 set contact-info "FortiSIEM Admin extension 8024"
 set location "Server Farm"
end

config system snmp user
edit "fortisiem_user"				>>> or choose your preferred SNMPv3 username here
 set status enable
 set queries enable
 set security-level auth-priv
 set auth-proto sha
 set auth-pwd "P@ssw0rd@123"			>>> sha password
 set priv-proto aes
 set priv-pwd "P@ssw0rd@123"			>>> aes password
 set notify-hosts "192.168.100.145"
 next
end

2/ Setup in FortiSIEM
---------------------
2.1/ Admin > Setup > Credentials (Device Type: Generic, Access Protocol: SNMP v3, Security Name: fortisiem_user or your previously chosen SNMPv3 username) 
		                 > IP-Range-to-Credential-Associations
			         > Test
2.2/ Admin > Setup > Discovery
- Range Scan - sequentially discover each device in one or more IP ranges and CIDR subnets.
- Smart Scan - first discover the Root IP & list of devices it knows, then discover each of these devices & list of devices they know about until exhausted
- AWS/Azure Scan - discover the devices in AWS/Azure Cloud learnt via AWS/Azure SDK
- L2 Scan - FortiSIEM will discover only the Layer 2 connectivity of the devices.
- Nozomi Scan - FortiSIEM will discover the devices in Nozomi SCADAguardian and CMC learnt via Nozomi REST API

Protocol: SSH
--------------
Used for: Performance Monitoring, Security and Compliance
---------------------------------------------------------
Information Discovered: Running configuration
---------------------------------------------
Metrics collected: Configuration Change
=======================================
Add these two lines to /etc/ssh/ssh_config of the FortiSIEM collector so its SSH Client prefer password authentication over public key authentication when connecting to the ForiGate:
	PreferredAuthentications password
	PubkeyAuthentication no

Protocol: Syslog
--------------
Used for: Availability, Security and Compliance
-----------------------------------------------
Information Discovered: Device type
-----------------------------------
Metrics collected: All traffic and system logs
==============================================
config log syslogd setting
    set status enable
    set server "192.168.100.155"
    set facility user
    set port 514
end

Protocol: Netflow
--------------
Metrics collected: Firewall traffic, application detection and application link usage metrics
=============================================================================================
config system interface
edit port1
set netflow-sampler both
end

config system netflow
set collector-ip "192.168.100.155"
set collector-port 2055
end

Display application information
-------------------------------
Go to Policy & Objects > Firewall Policy > Policy > Security Profiles (SSL inspection + App Control)

Discovery Settings
-------------------
Before you initiate discovery, you should configure the Discovery Settings in your Supervisor as required for your deployment
ADMIN > Settings > Discovery > Generic, 
			     > Device Filter, 
			     > Application Filter, 
			     > Location (set location information for devices in CMDB. Update locations of multiple devices with private IP addresses only)
			     > CMDB Groups (write rules to add devices in CMDB Device Group and Business Service Groups of your choice)

Resources
----------
FortiSIEM 6.7.4 External Systems Configuration Guide: https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/780675/fortisiem-external-systems-configuration-guide-online

Fortinet FortiGate Firewall:
https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/751381/fortinet-fortigate-firewall

Fortinet FortiGate Firewall - What is Discovered and Monitored:
https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/751381/fortinet-fortigate-firewall#What

Configuring SNMP v3 on FortiGate:
https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/751381/fortinet-fortigate-firewall#Configur7

Configuring SSH on FortiSIEM to communicate with FortiGate:
https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/751381/fortinet-fortigate-firewall#Syslog

Configuring FortiSIEM for SNMP and SSH access to FortiGate
https://docs.fortinet.com/document/fortisiem/6.7.4/external-systems-configuration-guide/751381/fortinet-fortigate-firewall#Configur

Online Help TOC:
https://help.fortinet.com/fsiem/6-7-4/Online-Help/HTML5/Home.htm

Discovery Settings:
https://help.fortinet.com/fsiem/6-7-4/Online-Help/HTML5_Help/Discovery_Settings.htm

Setting Credentials:
https://help.fortinet.com/fsiem/6-7-4/Online-Help/HTML5_Help/Setting_Credentials.htm

Discovering Devices:
https://help.fortinet.com/fsiem/6-7-4/Online-Help/HTML5_Help/Discovering_devices.htm

Working with Custom Properties:
https://help.fortinet.com/fsiem/6-7-4/Online-Help/HTML5_Help/Creating_Custom_Properties.htm
```


###### SNMP

![[FortiSIEM U/Section 02 - Device Discovery/002.png]]

**How to setup the SNMP in FortiGate?**

- we need to configure the interface that will receive the SNMP polling from your FortiSIEM collector to allow access to the SNMP
	- we know that firewall is a very critical device and each and every protocol should be allowed explicitly, otherwise, the firewall will drop this communication natively by default.
- so in order to allow the FortiSIEM to pull the FortiGate and to get the metrics mentioned above, so first thing you need to do is to allow SNMP traffic


- so normally both FortiSIEM and FortiGate will be in the same VLAN or same subnet
- so basically FortiSIEM collector is the one that will be using to polling to the FortiGate, in fact, you can use FortiSIEM supervisor itself to do the polling, but it's preferable to use the FortiSIEM collector, because in this way you can offload the performance monitor to FortiSIEM collector and leave the supervisor for the other task like UI, managing the database, worker functionality

- open the FortiGate, to enable the SNMP

![[FortiSIEM U/Section 02 - Device Discovery/003.png]]

- you can enable it here

![[FortiSIEM U/Section 02 - Device Discovery/004.png]]

- or you can use the FortiGate administrative commands mentioned in the above text

- you can try this in Lab, you can download the FortiGate virtual machine from the fortigate support and use it in your lab


- note that fortigate has another interface. this interface is the inside interface and this is connected to some windows machine in the above picture

- note that the command mentioned earlier is not accurate because if we apply the command, it will override the configuration that is currently set. so the current configuration contains PING, HTTPS, etc rules, so if we put SNMP only through commands, it will override and actually we will loose the access.
- if you only mention the snmp
```bash
config system interface
edit "port1"
 show
 set allowaccess snmp
end
```
- so check what we have on FortiGate and then add SNMP as well.

![[FortiSIEM U/Section 02 - Device Discovery/005.png]]

- now update with the snmp and you are good to go. now we didn't loose the access

```bash
config system interface
edit "port1"
 show
 set allowaccess ping https ssh http fgfm snmp
end
```
- to verify
```bash
show system interface
```
- now we need to enable the SNMP itself

```bash
config system snmp sysinfo
 set status enable
 set description "FortiSIEM collector polling inbound on interface port 1"
 set contact-info "FortiSIEM Admin extension 8024"
 set location "Server Farm"
end
```
- to verify
```bash
show system snmp sysinfo
```
- now we need to create some SNMP user to be used by the FortiSIEM.
- here we are enabling the SNMP version 3 because, SNNPv1, SNMPv2 is not secure enough 

```bash
config system snmp user
edit "fortisiem_user"				# >>> or choose your preferred SNMPv3 username here
 set status enable
 set queries enable
 set security-level auth-priv
 set auth-proto sha
 set auth-pwd "P@ssw0rd@123"			# >>> sha password
 set priv-proto aes
 set priv-pwd "P@ssw0rd@123"			# >>> aes password
 set notify-hosts "192.168.100.145"
 next
end
```

- At the end, we need to put the IP of the FortiSIEM collector, as we said FortiSIEM collector will be doing the polling functionality to offload this task from the supervisor
- to verify
```bash
show system snmp user
```
- until this point, we setup for the FortiGate, now we need to setup for the FortiSIEM

---

- go to `FortiSIEM` > `ADMIN > Credential` > `Step1: Enter Credentials > New`

*Remember:*
- whatever we setup in FortiGate will be used in FortiSIEM
- you can choose more secure algorithm for encryption, if the external device is supporting that
	- in our case FortiGate is currently supporting the SHA only

![[FortiSIEM U/Section 02 - Device Discovery/006.png]]

- `Step2: Enter IP Range to Credential Associations > New`
	- this is more or less similar to template that we defined for the windows and linux agent where we defined some template and then associated that template to some device.
	- so here we define credentials and associate the credentials to some IP range
		- the IP is FortiGate IP

![[FortiSIEM U/Section 02 - Device Discovery/007.png]]

- then do the test in `Step2: Enter IP Range to Credential Associations > Test > Test connectivity`
	- so if the host is reachable, then it will attempt to make the SNMP access using credentials

- if its showing `Test Connectivity Failed` check `Discovered by _______`
	- in our case, we should choose `Discovered by FortiCollector`, because our configuration is by FortiCollector
- sometime it will be misleading, other way to test is start the `tcpdump` in the FortiCollector

![[FortiSIEM U/Section 02 - Device Discovery/008.png]]

- Here we can see that the communication is happening in both direction

- now let's try to start the discovery
- go to `FortiSIEM` >`ADMIN > Discovery > New`
- there are many scans you can choose,
		- `Range Scan`: here you basically provide an IP or a range, then it will be sequentially discovered
		- `Smart Scan`: you provide only one IP (root IP, it's like a router and it has some IP's in its IP table)
			- so it knows about the neighbors, and it knows about other IP's
			- so this root IP will actually provide list of devices to the FortiSIEM, then the FortiSIEM will start scanning this list of devices one by one, and each device can provide another list, and the process will continue until the list of devices is exhausted and every IP has been scanned
		- `Nazomi Scan`: Nazomi is a monitoring solution for the OT environment, so the FortiSIEM can integrate with the scanner guardian of the Nazomi System to actually get the information about the OT environment through REST API, because the OT is very sensitive environment, it has its proprietary protocols, its not using the normal IT protocols.
			- so basically we are actually interfacing with some system that's able to understand this OT environment which is `Nazomi` in our case, this is the very good feature in the FortiSIEM (you can't find this SIEM solution in other SIEM, where it can provide you with a visibility of the OT network or the OT infrastructure)

- In our case, we can use `Range Scan`

![[FortiSIEM U/Section 02 - Device Discovery/009.png]]


- now click on `ADMIN > Discovery > Discover` and don't forget to choose `Discover by FortiCollector2` in the right hand side

![[FortiSIEM U/Section 02 - Device Discovery/010.png]]

- here you can see its successfully discovered and it's monitoring all the system information

- now you can navigate to `CMDB` to see the devices being discovered, now change the status from `Pending` to `Approved` in `Action` tab
	- the `Approved` status has some meaning and it actually affects the configuration, and its very important to change the status to `Approved`

- we can see the grouping under CMDB grouping, we can see this in summary tab

![[FortiSIEM U/Section 02 - Device Discovery/011.png]]

- note that the `configuration` (can be found in `Advanced` tab in above picture) is not something that is retrieved by SNMP, we need SSH


- in the left hand side of `DASHBOARD` you can choose `FortiSIEM Dashboard`
	- this has the widget for the `Performance Event Types`