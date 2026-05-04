## Product Key

- show original Product Key (stored in UEFI BIOS; could differ from the one currently in use)

```bat
wmic path softwarelicensingservice get OA3xOriginalProductKey
```

```ps1
(Get-WmiObject -query 'select * from SoftwareLicensingService').OA3xOriginalProductKey
# or (Get-CimInstance -ClassName SoftwareLicensingService).OA3xOriginalProductKey
```

- show DigitalProductId stored in registry

```bat
powershell -Command "$d=(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name DigitalProductId).DigitalProductId; $c='BCDFGHJKMPQRTVWXY2346789'; $k=''; for($i=24;$i -ge 0;$i--){ $cur=0; for($j=14;$j -ge 0;$j--){ $cur=$cur*256 + $d[$j + $i*15] } for($x=0;$x -lt 25;$x++){ $k = $c[$cur % 29] + $k; $cur = [math]::Floor($cur/29) } $k = $k.Substring(1) } $k = $k -replace '(.{5})','$1-'; $k.TrimEnd('-');"
```

- verify activation status

```bat
slmgr.vbs /dlv
```
