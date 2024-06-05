- it will be in `RESOURCES > Reports`, and there will be multiple sub groups

![[021.png]]

- so create report, it's better to actually create specific folder for your customer - this will ease the managing of your resources

- to create report, first go to newly created folder and click on `New`
- you will have 3 steps

- Step 1: General - here you need to select the `Group`, and also need to check if its a `Anomoly Detection Baseline` or not

![[022.png]]

- Step 2: Define Condition - this is very similar to condition we define inside the Rule
	- for example, if you go the `ANALYTICS` tab, and check the logs, so based on events we can design the report based on some criteria like `Event Type`

![[023.png]]

- Step 3: Define Display Column - here we can display things interesting for us

![[024.png]]

- and save the report

- you will se more details in summary tab by clicking `^`, you will see two tabs
	- `Summary`
	- `Schedule` - `Definition` and `Results`

- you can define the schedule for your report - here you need to define `Report Time Range`
	- Report time could be `Relative` or `Absolute`
	- `Absolute` is for if you are doing some threat hunting exercises or if you are looking for specific time window

- you can also define the trend

![[025.png]]

- Trend Interval
	- `Auto` - means it will stick to the query or if your `Relative` time is bit long (like month or weeks), in this case, the trend is useful to actually tell the report to take the each hour/day/week
		- if you are not sure about the trend just leave it auto

- next
![[026.png]]

- next - you need to define the format of the report (`PDF`, `CSV`, `RTF`)

![[027.png]]

- you can run the report manually

![[028.png]]

- after running manually

![[029.png]]

- this report can be used in the dashboard

![[030.png]]

![[031.png]]

- there is a faster way to create the report from your search result itself instead of creating report from scratch

![[032.png]]

![[033.png]]

- now you can use the search result saved from the `ANALYTICS` page, this will give the exactly same output

![[034.png]]

![[035.png]]







