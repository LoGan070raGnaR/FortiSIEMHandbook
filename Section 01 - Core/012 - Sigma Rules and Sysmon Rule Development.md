
- [Sigma rules adopted in FortiSIEM](https://docs.fortinet.com/document/fortisiem/6.2.0/release-notes/498610/new-features#Public)
- [Sigma rules on Github](https://github.com/SigmaHQ/sigma/tree/master/rules)

- How you can convert `sigma` rules into `FortiSIEM` rule?

- Many rules are adopted from sigma rules in FortiSIEM. for example,
	- [Windows: Windows Powershell Web Request](https://github.com/SigmaHQ/sigma/blob/master/rules/windows/powershell/powershell_script/posh_ps_web_request_cmd_and_cmdlets.yml) this can be found in `FortiSIEM` > `RESOURCES` > `Rules` and search for it in search bar.
	- to get the name refer this [link](https://docs.fortinet.com/document/fortisiem/6.2.0/release-notes/498610/new-features#Public)
	- you can edit the rule with `Edit` option

##### Comparison between Sigma and FortiSIEM rule

![[FortiSIEM U/Section 01 - Core/007.png]]

- we can see here, some of them are adopted by Fortinet in FortiSIEM rules
- it's not exact one to one mapping, they just use it for partial case, the sigma rule they referred is general, here they are using it for specific use case

![[FortiSIEM U/Section 01 - Core/008.png]]

##### Developing our own rule

- let's create the one that will actually utilize the parent command line and the parent process and also the fields for the file name or the software that is being triggered by this parent process to check for malicious behavior.
- first let's start with simple rule, as powershell command or powershell as the parent process and the commands issued by interpreter of powershell (which is odd case, because powershell shouldn't be used to issue executable like notepad or to open the winword something like it)
- here we will try to how to utilize sysmon logs and utilize the different fields inside it to build your own use cases.

##### Goal:

- To simulate our use case to trigger any executable from the powershell and check the logs in sysmon logs and try to write a rule to give us an incident based on that
- just execute any executable (e.g., calc.exe) in the powershell.

![[FortiSIEM U/Section 01 - Core/009.png]]

- now check the `ANALYTICS` tab to check the log

![[FortiSIEM U/Section 01 - Core/010.png]]

- and the parent process is powershell

![[FortiSIEM U/Section 01 - Core/011.png]]

- first make sure that logs are logging then click on raw event log

![[FortiSIEM U/Section 01 - Core/012.png]]

- then look for some interesting field

![[FortiSIEM U/Section 01 - Core/013.png]]


![[FortiSIEM U/Section 01 - Core/014.png]]

- these are the two interesting fields that we can check

##### To create the rule

- click on `RESOURCES` > `New`
- Step1: General
	- Group - Most of the time `security`
	- Rule Name - to distinguish from the default rule, add some prefix
	- Description
	- Event Type - it will be `Rule Name` with underscore
- Step2: Define Condition
	- here you need to add time window
	- and you can add filters (in Subpattern section)

![[FortiSIEM U/Section 01 - Core/015.png]]

- from the sysmon log we can find the the value of the Attribute

![[FortiSIEM U/Section 01 - Core/016.png]]

![[FortiSIEM U/Section 01 - Core/017.png]]

- You can check and validate value in `Express on Builder` option. and note that sometimes the `spaces` matters
- now we can add for parent process

![[FortiSIEM U/Section 01 - Core/018.png]]

- and the value should be `powershell.exe`

![[FortiSIEM U/Section 01 - Core/019.png]]

- we can also use the `regular expressions (REGEXP)` for complex filters

![[FortiSIEM U/Section 01 - Core/020.png]]

*Note:*
- `REGEXP` will allow you to account for all the path syntax (like `\\` for account for the backward slash (`\`))
![[FortiSIEM U/Section 01 - Core/021.png]]

- Also we need to look for any executable in the `Command` (here `calc.exe`)

![[FortiSIEM U/Section 01 - Core/022.png]]

- And also software being called

![[FortiSIEM U/Section 01 - Core/023.png]]

- we can `Run as Query` against historical data

![[FortiSIEM U/Section 01 - Core/024.png]]

- output

![[FortiSIEM U/Section 01 - Core/025.png]]

- Step3: Define Action
	- at least one event attribute in `Action` section

![[FortiSIEM U/Section 01 - Core/026.png]]

- finally save and review

- activate the rule and try to simulate the attack, and test the rule

- to test the rule:
	- go to `ANALYTICS` tab, and look for the exe file you have runned
	- in `INCIDENT` section, we should receive one `Medium` severity incident
		- if it's not showing check in `Rules` tab in `RESOURCES` section if there is any `Sync Error`
			- if there is check the original system which is running `FortiSIEM` for our rule
			- tail for `phoenix.log`
			```bash
			tail -f /opt/phoenix/log/phoenix.log | egrep -i 'PHL_ERROR|OUR_RULE_PREFIX'
		 ```
		 - also you can `Run as Query` against historical events

- now you can see the rule triggering in `INCIDENT` section

![[FortiSIEM U/Section 01 - Core/027.png]]

- here is the rule summary

![[FortiSIEM U/Section 01 - Core/028.png]]

