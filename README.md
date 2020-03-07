# Collecting Files from Endpoints via McAfee ePolicy Orchestrator

## Requirements

### Products Versions
This procedure will work only if the minimum version listed bellow for each components are respected:
- ePO Version: 5.9.1 (ePO On-Premise or Hosted on AWS/Azure only)
- McAfee Agent Version: 5.6.X

### Permission Set
You will have to use the **Single System Troubleshooting** feature from ePO, so your user account must have enough rights to proceed. If this line stay greyed in the **System Tree** menu, ask your ePO Administrator for adapted **Permission Set**.<br>
If you are not familliar with collecting **McAfee Agent** log, please review the **Knowledge Center** article [KB91283](https://kc.mcafee.com/corporate/index?page=content&id=KB91283). Or review the McAfee Agent 5.6 documentation chapter [Viewing McAfee Agent logs](https://docs.mcafee.com/bundle/agent-5.6.x-product-guide/page/GUID-F2605AD0-BCFC-415A-8A56-212526CC4324.html).

### McAfee Agent Policy
The **General** settings of your McAfee Agent policy must be modified to permitted the following actions to write files in the McAfee Agent logs folder. To do this, please unselect the setting __Enable self protection (Windows Only)__ from the **General options** section. To re-enforce protection of the McAfee Agent services, process, files and registry we will have to implement a custom set of **Expert Rules** through Endpoint Security Exploit Prevention.
Please see Appendix A to implement a proposed set of rules.

## Preparing the file for collect

You can prepare the file for collect from **McAfee ePO** through a custom package or from **MVISION EDR** through a custom reaction. Feel free to use the most convenient for you.

### Preparing the file for collect from MVISION EDR

#### Custom Reaction Creation
We need first to create a custom reaction to run on a targeted system with the file path in parameter to prepare that file for collect.

From your MVISION EDR console, go to Menu -> Catalog -> Reaction -> and click Add. Define the custom reaction as follow:
- **Name:** _PrepareFileForCollect
- **Description:** Prepare the file for collect.
- **Windows Type:** Execute OS Command
- **Content:** Use the following script.
```bat
@ECHO OFF
SETLOCAL
SET FILEPATH={{filepath}}
SET MYKEY=HKLM\SOFTWARE\McAfee\Agent
FOR /F "skip=2 tokens=2*" %%A IN ('REG QUERY %MYKEY% /v "DataPath"') DO SET DATAPATH=%%B
COPY /Y "%FILEPATH%" "%DATAPATH%logs"
ENDLOCAL
EXIT /B %ERRORLEVEL%
```
- **Reaction Arguments:** Click Add.
- **Name:** filepath
- **Type:** String
- **Timeout:** You can let the Timeout to the default 60 sec value or adjust the value if you want to let more time for the local copy of the file.

Click Save to finish the creation of the custom reaction.

#### Use case A: Collect a file after a search
Let's say you did a search for a suspicious file with a **Real-time Search** query like this:
```
HostInfo hostname and Files full_name, md5 where Files name contains "Sample.exe" and Files status equals "current"
```
Select the system where you want to collect the file, copy the file path and click on Actions -> Custom -> _PrepareFileForCollect then paste the file path. Click OK to copy the file for collect. Repeat this action for each file you want to collect.

#### Use case B: Collect a process dump
Let's say you did a search for a suspicious running process with a **Real-time Search** query like this:
```
HostInfo hostname and Processes id, name where Processes name contains "notepad"
```
Select the system where you want to collect the file, copy the process id and click on Actions -> Investigation -> Dump Process To File, then paste the process id and type a file name prefixed with the following path:
```
C:\ProgramData\McAfee\Agent\logs\
```
i.e.: C:\ProgramData\McAfee\Agent\logs\pid2556.dmp. Click OK to dump the process to a file ready for collect.

#### Note
If you want to follow the progression of your actions or review the one that have been taken previously, click Menu -> Action History.

### Preparing the file for collect from McAfee ePO

#### Checkin the package
TBD
#### Define the Deployment Client Task
TBD
#### Run now the task
TBD

## Collecting the files
From your ePO Console, click Menu -> System Tree, then use the filter or select the targeted Organisation Unit within your system tree to identify your targeted system.

Select the system, click Actions -> Agent -> Single System Troubleshooting, then click Collect and wait while the McAfee Agent is sending all files to ePO DB until you get "Logs are received successfully". Then click the Download button and save the zip archive on your local system.

Unzip the archive to retreive targeted files.<br>
Enjoy! ;-)

## Appendix

### Appendix A - Prevent modification of McAfee Agent
TBD

### Appendix B - Cleaning files for collect
TBD
