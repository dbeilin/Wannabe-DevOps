![](images/powershell/shutdown-delete1.png)

This is a simple script to shutdown and delete multiple VMs at once in VMware.

Make sure you have a list of VMs in a text file. It’s simple to export a list of VMs in a certain folder in your vcenter, but just as an example, it should look like this:

![](images/powershell/shutdown-delete2.png)

Then, here’s the script, run it:

```powershell
$vcenter = Read-Host("Enter VCenter IP")
$VMs = (Get-Content C:\temp\vmlist.txt)
$vmObj = Get-vm $vms
Connect-VIServer $vcenter
 
foreach($active in $vmObj){
if($active.PowerState -eq "PoweredOn"){
Stop-VM -VM $active -Confirm:$false -RunAsync | Out-Null} 
}
Start-Sleep -Seconds 7
  
foreach($delete in $vmObj){
Remove-VM -VM $delete -DeleteFromDisk -Confirm:$false -RunAsync | Out-Null}
```

You can separate this script to just shutdown or just delete from disk. Use it whatever way you please.
It’s a simple script, but as always, if I used it at least a few times, it worth sharing 🙂