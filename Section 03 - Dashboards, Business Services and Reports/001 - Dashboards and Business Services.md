##### DASHBOARD

- `DASHBOARD` is very important utility for any SOC environment where the monitoring team being level one or whatever the structure of your SOC is always watching the dashboard and they can identify anomalies based on some spike in some trend or some values that is not normal

- another important feature in FortiSIEM is `Business Services` - it's a way of grouping your mission critical devices/applications into some groups and these groups can be added to a dashboard called Business Service Dashboard
	- so the monitoring team can easily focus on the mission critical services in the environment and have specific dashboard for them and focus their monitoring.

- on the left hand side you can see the dashboard folder, for each folder you will have separate dashboards
	- and each dashboard will have multiple widgets, and each widgets will display the data in different chart types
- in the gear icon you can modify the content or settings of the widget

![[001.png]]

- you can create your own dashboard in the Dashboard folder by selecting `New` in the far left side inside the folder
- then click on `+` inside newly created dashboard, name the widget and select the `Type`, default will be `Widget Dashboard`
	- so basically when you add the widget, you are adding the reports
	- so when you click on `+` on the newly created widget, you will see two types of report
		- `Reports`
		- `CMDB Reports` - this is basically for the discovered devices in your environment by the discovery feature

- one example to create the widget
	- after finding two types of report, go to `Reports > Functions > Security > Authentication > Top Devices By Failed Login` and then select right arrow (`>`)
	- this will bring the widget to the dashboard

![[FortiSIEM U/Section 03 - Dashboards, Business Services and Reports/002.png]]


##### Business Services

- go to `CMDB` > `Business Services`
	- here you will see some of the out of the box services
	- for example `Operational Technology`

![[FortiSIEM U/Section 03 - Dashboards, Business Services and Reports/003.png]]

- so basically you can utilize this nice grouping of the resources to define your mission critical systems and then put them into a dashboard and have your SOC focusing on the critical infrastructure
- in the above figure, we can see that the`Members` are zero, so during Discovery this will be populated or you can add devices manually in `Edit` section of `CMDB > Business Services > Operational Technology`
- you can create your own group here also

- after this you can go to `DASHBOARD` to include the Business Service into Dashboard
	- `Create New Dashboard` by clicking `+` icon in the right hand side

![[004.png]]

- then click on the hard disk like icon to add your Business services

![[005.png]]

- if you want to delete the custom services, it's better to delete the dashboards you created or depending resources first

---
**How to define your custom business services and how you can also let the FortiSIEM automatically categorize it  into your custom business services

![[006.png]]

- inside this group create `New Business Service` by clicking `New`

![[007.png]]



---
- now let's do the scenario where we will simulate the reply logs to FortiSIEM for two mission critical applications
- we will rely on log mechanism of the FortiSIEM to discover these two devices and create host for them

- let's reply the events for two IPs

```bash
for i in {1..30}; do echo "<85>Oct 20 11:16:23 10.10.50.50 sudo: pam_unix(sudo:auth) : authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=dieter rhost= user=dieter" | nc 192.168.100.155 514; done
```

- one echo is enough

```bash
do echo "<85>Oct 20 11:16:23 10.10.50.50 sudo: pam_unix(sudo:auth) : authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=dieter rhost= user=dieter" | nc 192.168.100.155 514
```

- here we are simulating that device is sending an authentication failure and then we pipeline the output to our FortiSIEM supernode on port 514

- now check the `ANALYTICS` tab to view the log

![[008.png]]

- the attributes in the `Event Details`, we can define in the discovery settings, so the FortiSIEM can automatically locate these devices into the group

![[009.png]]

- go to `ADMIN > Settings > Discovery > CMDB Groups > New`, depending on the `Event Details` fill the information

![[010.png]]

- this is the dynamic way of allocating the discovered resources into the business service group

- now go back to `CMDB > Business Services > <Created Service>` and then refresh to populate the `Members` tab
	- this will not work until we send the another log

![[011.png]]

- for more details click on `Edit` tab

![[012.png]]

- now let's add this business service into our new dashboard
- go to `DASHBOARD > +`

![[013.png]]

- then add the Business service to the dashboard

![[014.png]]

- view the dashboard

![[015.png]]

- so for no devices is impacted inside this newly created business services so for


- let's simulate the attack against this business services
- let's create the rule to detect the bruteforce attack

![[016.png]]

- the subpattern we included

![[017.png]]

- this is what we are checking

![[018.png]]

- let's simulate the attack 
	- for 10.10.50.50 and 10.10.50.51

```bash
for i in {1..30}; do echo "<85>Oct 20 11:16:23 10.10.50.50 sudo: pam_unix(sudo:auth) : authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=dieter rhost= user=dieter" | nc 192.168.100.155 514; done
```

```bash
for i in {1..30}; do echo "<85>Oct 20 11:16:23 10.10.50.51 sudo: pam_unix(sudo:auth) : authentication failure; logname= uid=1000 euid=0 tty=/dev/pts/3 ruser=dieter rhost= user=dieter" | nc 192.168.100.155 514; done
```

- now we can see the logs are coming and the top 5 impacted devices will also populate

![[019.png]]

- here you can see our two hosts are populated here 
	- it's displaying first part of the name, we can actually rename this to whatever we want
	- we can confirm this in `INCIDENTS` tab

![[020.png]]

- it will also trigger the rule we created, it should be in `INCIDENTS` tab in `Top Impacted Hosts - By Severity / Risk Score`

- this is the beauty of the dynamically assigning your mission critical devices or applications into dynamic groups, in this dynamic groups you can design the dashboard and you can drill down to see more details about your business service as well
