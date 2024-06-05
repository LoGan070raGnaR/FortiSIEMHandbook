- `syslog` protocol which allow us to collect all the traffic or all the system logs from the device (in our case FortiGate)
- configuration is very simple, we can do it in CLI, or we can do it in the FortiGate interface

- this is the configuration we applied at the FortiGate side

![[FortiSIEM U/Section 02 - Device Discovery/017.png]]

- here you can control which events you need to send to FortiSIEM or don't need to send

![[FortiSIEM U/Section 02 - Device Discovery/018.png]]

- same way you can configure the `syslog` in `Log & Report > Log Settings` section in FortiGate

![[FortiSIEM U/Section 02 - Device Discovery/019.png]]

- note that you need to provide the IP of the syslog server
- however more granular configuration can be done via CLI, here we can control the format, we can control the port

- CLI version
	- first we need to enable the syslog daemon, then we need to define the server IP (here, we use the supervisor IP, we will not upload the logs to the collector, the logs will be send directly to the supervisor)
		- only in the case of agent logs or the performance metrics we need to upload to the collector
	- facility can be chosen from the list of options 

![[FortiSIEM U/Section 02 - Device Discovery/020.png]]

- here in facility we have multiple options, you can choose among all of this

![[FortiSIEM U/Section 02 - Device Discovery/021.png]]

- here you can see the default format, we will not change anything
	- don't change to csv or cef, otherwise the logs will not be parsed correctly on the FortiSIEM

![[FortiSIEM U/Section 02 - Device Discovery/022.png]]

- steps:
```bash
config log syslogd setting
    set status enable
    set server "192.168.100.155"
    set facility user
    set port 514
end
```

- to verify 

![[FortiSIEM U/Section 02 - Device Discovery/023.png]]

- you can also verify from the FortiSIEM interface in `ANALYTICS` tab
	- performance events will be starting with `PH_DEV_MON`

![[FortiSIEM U/Section 02 - Device Discovery/024.png]]

- and also see in the above figure, the device events like FortiGate events are starting with normal names like `FortiGate`

