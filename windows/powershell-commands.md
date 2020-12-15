# Powershell commands

## Tab completion for powershell

```text
Set-PSReadlineKeyHandler -Key Tab -Function Complete
```

#### Search History

```text
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward
```

```text
#Get the list of log types
Get-EventLog -List

# Get logs which contains text vector
Get-EventLog "Application" | where Message -Like "*vector*"

#Get security type logs
Get-EventLog "Security" | where Message -Like "*vector*"

```

#### Shortcuts

```text
Get-Help [help]

Get-Command [gcm]
Get-Command *-service*

Invoke-Command [icm]
Invoke-Command -ScriptBlock {Get-EventLog system -Newest 50}

Invoke-Expression [iex]
Invoke-Expression $Command

Invoke-WebRequest [iwr]
WebRequest -Uri "https://docs.microsoft.com").Links.Href

Set-ExecutionPolicy
Set-ExecutionPolicy -ExecutionPolicy Restricted

Get-Item [gi]
Get-Item M*

Copy-Item [copy]
Copy-Item "C:\Services.htm" -Destination "C:\MyData\MyServices.txt"

Remove-Item [del]
Remove-Item "C:\MyData\MyServices.txt"

Get-Content [cat]
Get-Content "C:\Services.htm" -TotalCount 50

Set-Content [sc]
Get-Content "C:\Services.htm" -TotalCount 50 | Set-Content "Sample.txt"

Get-Variable [gv]
Get-Variable -Name "desc"

Set-Variable [set]
Set-Variable -Name "desc" -Value "A Description"

Get-Process [gps]
Get-Process *explore*

Start-Process [saps]
Start-Process -FilePath "notepad" -Verb runAs

Stop-Process [kill]
Stop-Process -Name "notepad"

Get-Service [gsv]
Get-Service | Where-Object {$_.Status -eq "Running"}

Start-Service [sasv]
Start-Service -Name "WSearch"

Stop-Service [spsv]
Stop-Service -Name "WSearch"

ConvertTo-HTML
Get-Service | ConvertTo-HTML -Property Name, Status > C:\Services.htm

```

```text
PS C:\> Start-Job -ScriptBlock {Get-Process}
Id     Name       PSJobTypeName   State       HasMoreData     Location             Command
--     ----       -------------   -----       -----------     --------             -------
1      Job1       BackgroundJob   Failed      False           localhost            Get-Process

PS C:\> (Get-Job).JobStateInfo | Format-List -Property *
State  : Failed
Reason :

PS C:\> Get-Job | Format-List -Property *
HasMoreData   : False
StatusMessage :
Location      : localhost
Command       : get-process
JobStateInfo  : Failed
Finished      : System.Threading.ManualReset
EventInstanceId    : fb792295-1318-4f5d-8ac8-8a89c5261507
Id            : 1
Name          : Job1
ChildJobs     : {Job2}
Output        : {}
Error         : {}
Progress      : {}
Verbose       : {}
Debug         : {}
Warning       : {}
StateChanged  :

PS C:\> (Get-Job -Name job2).JobStateInfo.Reason
Connecting to remote server using WSManCreateShellEx api failed. The async callback gave the
following error message: Access is denied.
```

{% embed url="https://docs.microsoft.com/en-us/powershell/scripting/samples/sample-scripts-for-administration?view=powershell-7.1" %}

{% embed url="https://docs.microsoft.com/en-us/powershell/scripting/samples/managing-processes-with-process-cmdlets?view=powershell-7.1" %}

{% embed url="https://docs.microsoft.com/en-us/powershell/scripting/samples/managing-services?view=powershell-7.1" %}



