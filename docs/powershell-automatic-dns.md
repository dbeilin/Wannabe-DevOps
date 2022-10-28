![](images/powershell/dns/dns.png)

If you need to add multiple A records to your DNS, here’s a quick way to do so.

Create a CSV file, name it however you like (make sure you correct the name of the file in the script itself).
Then, the first row should be filled with the words “Hostname” and “IP”. You can change that as wel but once again, make sure you edit the script accordingly.

![](images/powershell/dns/excel.png)

Then, edit this script with your relevant information like the CSV filename, zone name, and your DC.

```powershell
$IPs = Import-Csv "C:\temp\dnsIPs.csv"
foreach ($item in $IPs)
{
    $hostname = $item.("Hostname")
    $IP = $item.("IP")
    Add-DnsServerResourceRecordA -Name $hostname -ZoneName "yourcorp.com" -IPv4Address $IP -ComputerName "DC1" -AsJob
    Get-Job | Stop-Job
}
```

That’s it, now run it and watch it go 🙂