
- note that `syslog` is just providing events that are happening on your FortiGate like
	- interface up/down
	- user has logged in/out
	- some connection is initiated or triggered
	- some connection closed
- syslog is just the events that has occured.

- `Netfow` will provide more information about the flow itself
	- it tells you source and destination
	- protocol
	- port being used
	- application detection - for this we need to make some extra configuration
	- usage
	- duration of the session

- `Netflow` is useful if you need to
	- detect scans
	- lateral movement
- the syslog will not help you in this scenarios. that's why netflow is important

- first we need to enable the netflow sampling on the interface, the one facing the FortiSIEM

```bash
config system interface
edit port1
set netflow-sampler both
end
```
- also we need to configure the reporting IP, which is the collector IP or the supervisor IP - here we are configuring the netflow itself to send the traffic to supervisor

```bash
config system netflow
set collector-ip "192.168.100.155"
set collector-port 2055
end
```

- the remaining part is to enable the FortiGate itself to intercept the SSL traffic and to apply application control profile on the policy (the firewall policy), so its able to detect the application and send this information in the NetFlow
	- for this we need to edit the firewall policy and enable two security profiles (`SSL inspection + App control`)
- to do this go to FortiGate UI > `Policy & Objects > Firewall Policy`
	- in `Security Profiles`

![[FortiSIEM U/Section 02 - Device Discovery/025.png]]

![[FortiSIEM U/Section 02 - Device Discovery/026.png]]

- this completes the configuration for the Netflow

- check in `ANALYTICS` page, and check if the events are coming
- in order to distinguish between syslog and the netflow, you can modify the display fields and add the `External Event Receive Protocol` and `Apply & Run`, it will show in the separate column

##### Discovery Settings

- this is important because, from one deployment to another, you may have different setups, you may have different components in your infrastructure that requires you to make some filtration or some customization to your discovery
	- for example, you may choose to filter some devices that you don't need to include in your discovery
	- you may filter out some applications that you don't need to be discovered
	- you may choose to exclude some virtual IP's, for example
		- if you have high availability between two firewalls and they are using a virtual IP, if we don't exclude this virtual IP from discovery, it will be recognized as, these two firewalls will be recognized as one firewall 
		- and some servers are sharing their credentials for example if you are logging to your email, the email itself will use the credentials of Active Directory to authenticate you against the mailbox, so may be this will be discovered again as another entity,
			- so actually you need to exclude this mailbox, this proxy, this server, this shared credentials from the discovery

```text
Discovery Settings
-------------------
Before you initiate discovery, you should configure the Discovery Settings in your Supervisor as required for your deployment
ADMIN > Settings > Discovery > Generic, 
			     > Device Filter, 
			     > Application Filter, 
			     > Location (set location information for devices in CMDB. Update locations of multiple devices with private IP addresses only)
			     > CMDB Groups (write rules to add devices in CMDB Device Group and Business Service Groups of your choice)
```

- this can be found in ADMIN page
- in `ADMIN > Settings > Discovery > Generic` you will exclude the virtual IPs from the discovery
	- so the individual IPs of the devices will be used to identify the primary and the standby as the two separate entities
	- if you don't exclude the virtual IPs, the virtual IP will be seen once on the primary when it's primary and once on the secondary when it's taking the ownership
		- this will result in the confusion in the FortiSIEM, and they can be merged as one entity because the same IP has been seen twice on both entities, so they will be merged together
		- but if you excluded the virtual IP from the dicovery, then the individual IP will only be seen and the dicovery will be successful

- also you can `Exclude shared devices IPs`, the example is like mailbox, the proxy are using AD credentials to authenticate you, so this credential, if it seen by this devices or if this credentials are able to discover these devices, then the confusion will happen to the Analytics tab
- and if you have a situation like virtual appliances and they are using the same serial number again you need to exclude.


- as we previously said, in the CMDB, it is important to change the status to approved because if you didn't do so, in the `ADMIN > Settings > Discovery` settings, if you choose to trigger an instance from `Approved Device Only` then this has an effect.

![[FortiSIEM U/Section 02 - Device Discovery/027.png]]

- the default setting is `All Devices` regardless of the pending status or being approved, so any device with any status will be able trigger an event

##### CMDB Groups

- the CMDB groups, as we saw in, when we discovered the firewall, we get to know that firewall is grouped into 4 groups in the CMDB

![[FortiSIEM U/Section 02 - Device Discovery/028.png]]

- however, you can create your own group, just click on `+` symbol in the far left side and write some rule to assign the discovered device into that rule
- in `ADMIN > Disovery > CMDB Groups > New` , you can write your rule
- so assigning your discovered device into your custom groups or custom business service group (`Biz Services`) is not conflicting with the automatic assignment that the FortiSIEM is already doing
- in fortiSIEM, there are more than 70 custom properties you can use to tag your business assets or discovered infrastructure components

![[FortiSIEM U/Section 02 - Device Discovery/029.png]]

- so if you check the `Custom Propertie` you will not find any custom properties by default.
	- for this you need to go to `ADMIN > Device Support > Custom Properties`, from here you can see all the custom properties coming by default
	- for defining custom property click on `New`, then you have give `Name` and `Display Name`

![[FortiSIEM U/Section 02 - Device Discovery/030.png]]

- now go back to `ADMIN > Settings > Discovery > CMDB Groups > New > Custom Properties`

![[FortiSIEM U/Section 02 - Device Discovery/031.png]]

- if you check in the CMDB, you will notice that some assets actually coming with some attributes

![[FortiSIEM U/Section 02 - Device Discovery/032.png]]

- so basically you can find all the information in the [FortiSIEM External Systems Configuration Guide Online](https://docs.fortinet.com/document/fortisiem/7.1.6/external-systems-configuration-guide/780675/fortisiem-external-systems-configuration-guide-online) guide, where you can find what you need to request from your client, for example SSH credentials, SNMP credentials, AD service account or whatever to be able to pull or get the information into your FortiSIEM and build your CMDB database