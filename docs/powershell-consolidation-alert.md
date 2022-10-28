![](images/powershell/consolidation.png)

So you might have a few alerts already set thanks to a lot of public tools out there, but if you want to get an alert about a VM that needs consolidation without installing or buying anything, you can use a simple and short PowerShell script.

```powershell
Connect-VIServer -Server "your-vcenter"
$cneeded = Get-VM | Where-Object {$_.Extensiondata.Runtime.ConsolidationNeeded}
if (!$null -eq $cneeded) {
    Send-MailMessage -To "your.email" -From "from@email.com"  -Subject "Consolidation Needed!" -Body "$cneeded" -SmtpServer "server-here" -Port 25
}
```

That’s it. You can add it to a Task Scheduler and set the checks to happen whenever you like.
Hope this was somewhat helpful 🙂