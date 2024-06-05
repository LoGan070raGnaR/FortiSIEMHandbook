
- let's do some attack simulation against some file that contains important information (let's say, it is stored in the linux server)

![[FortiSIEM U/Section 01 - Core/033.png]]

- lets assume `VIP_File.txt` file contains some trade secret for one company
- so basically we need to have a way or some capability to detect if the file has been tampered with

- previously we have installed windows agent, here we will do the linux agent installation, this will allow us to enable FIM (File Integrity Monitoring) capability where we can detect such attack.
- thanks to FortiSIEM which comes with built in rules that can detect or check for events coming from the FortiSIEM agent where some files has been tampered with or some attributes has been changed in some file system.


- first thing is to install the FortiSIEM linux agent on linux server

##### Linux Agent Installation

```text
Prerequisites
-------------
1/ OS supported: CentOS/RHEL>7.4 and other linux flavors
2/ curl>7.19.7 & nss>3.36.0 otherwise >> yum update -y nss curl lib curl
3/ validate rsyslog >> systemctl status rsyslog.service >> if not running >> systemctl start rsyslog.service
4/ Make sure timzone is ok on the Linux machine >> timedatectl set-timezone Asia/Qatar
5/ The following packages must be installed (yum install <package_name>) before FortiSIEM Linux Agents can run on CentOS/RHEL:

libcap
audit
audispd-plugins
rsyslog
logrotate
If SELinux is enabled (sestatus), then the following packages also must be installed:
policycoreutils
libselinux-utils
setools-console

Install/Start/Configure the Agent
---------------------------------
1/ Download the Linux installation script from Fortinet Support website support.fortinet.com and upload it using WinSCP
2/ chmod +x fortisiem-linux-agent-installer-6.6.0.1633.sh
3/ Run the command: bash fortisiem-linux-agent-installer-<VERSION>.sh -s <SUPER_IP> -i <ORG_ID> -o <ORG_NAME> -u <AGENT_USER> -p <AGENT_PWD> -n <HOST_NAME>
bash fortisiem-linux-agent-installer-6.6.0.1633.sh -s 192.168.100.155 -i 1 -o super -u lnxusr -p P@ssword@123 -n LinuxSRV
4/ Start the agent: /usr/bin/sh -c /opt/fortinet/fortisiem/linux-agent/bin/phLinuxAgent &
5/ Define the template and assign to host
6/ If File Integrity monitoring is chosen, validate the SElinux context
ls -Z >> validate SE context is var_log_t >> or change >> chcon system_u:object_r:var_log_t:s0 VIP_File.txt

Log Rotation
-------------
https://docs.fortinet.com/document/fortisiem/6.6.3/linux-agent-installation-guide/201446/fortisiem-linux-agent#Log_Rotating__var_log_messages_to_Prevent_Filling_Up_the_Root_Disk

UnInstall the Agent
--------------------
/opt/fortinet/fortisiem/linux-agent/bin/fortisiem-linux-agent-uninstall.sh

Sources
-------
https://docs.fortinet.com/document/fortisiem/6.7.3/linux-agent-installation-guide/201446/fortisiem-linux-agent
https://help.fortinet.com/fsiem/6-6-0/Online-Help/HTML5_Help/Configuring_Linux_Agent.htm
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-working_with_selinux-selinux_contexts_labeling_files
https://unix.stackexchange.com/questions/79311/configuring-selinux-to-allow-logging-to-a-file-thats-outside-var-log
https://docs.fortinet.com/document/fortisiem/6.6.3/linux-agent-installation-guide/201446/fortisiem-linux-agent#Managing
```


- check the prerequisites

![[FortiSIEM U/Section 01 - Core/034.png]]

![[FortiSIEM U/Section 01 - Core/035.png]]

- also check if SELinux is enabled (sestatus)

![[036.png]]

- while installing the linux agent, we need to create the agent user in the `FortiSIEM` > `CMDB > Users > New`
	- make sure `System Admin` checkbox is checked and inside that select `Agent Admin` and put username and password
- retrieve server name from the linux server and then install

![[037.png]]

- to start the service

```bash
/usr/bin/sh -c /opt/fortinet/fortisiem/linux-agent/bin/phLinuxAgent &
```


##### Note: info about license usage and regarding the agents

- you will see separate count for number of `Agents` and number of `UEBA`
- actually `UEBA` is a feature inside the `Agent`
	- it's a kernel level component that is being installed along with the same agent that you install

![[038.png]]

- here we can see the usage.

![[039.png]]

- why we have UEBA because, in the installing the windows agent we have checked the UEBA

![[040.png]]

- this is how you can track your license usage


- now after successful installation of linux agent, its showing as `Registered` 

![[041.png]]

- to make it active (`Running Active`), we need to define the template

![[042.png]]

- in our case, we need to monitor important file (e.g., `VIP_File.txt` ), include this in FIM
	- `Push Files` will push the metadata into FortiSIEM (actually stored in SVN database)

![[043.png]]

- then we need to associate this template into some host

![[044.png]]

- and then apply, and then you can see `Running Active` in Health section in ADMIN panel


- let's check what are the built-in rules that can be useful to detect the modification attempt

![[045.png]]

- let's see what exact rule will be triggered when we try to modify the file

![[046.png]]

- you can view detailed event in `Event Details`

![[047.png]]

- you can also view details in `INCIDENTS` page in overview section

![[048.png]]

##### Note:

- One important capability of monitoring is that the ability to see the differences between the files being modified
- metadata of the files being modified is updated or uploaded to the SVN databases of the supervisors. so you can have the multiple revisions of the same file and you can compare between these revisions and pinpoint the changes that have occurred.
- you can also compare the metadata of the modified file against a master image. so you can also know what has been changed. to check this you have to check the record created of the host inside the CMDB database, from here you can check the file being modified in that host

![[049.png]]

- here we can see the modification in metadata. first we tried with file itself, the the file path
- you can select both and see the difference (useful when comparing two same files)

![[050.png]]

- this is the importance of FIM (File Integrity Monitoring)