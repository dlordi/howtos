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
@REM powershell -Command "$d=(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name DigitalProductId).DigitalProductId; $c='BCDFGHJKMPQRTVWXY2346789'; $k=''; for($i=24;$i -ge 0;$i--){ $cur=0; for($j=14;$j -ge 0;$j--){ $cur=$cur*256 + $d[$j + $i*15] } for($x=0;$x -lt 25;$x++){ $k = $c[$cur % 29] + $k; $cur = [math]::Floor($cur/29) } $k = $k.Substring(1) } $k = $k -replace '(.{5})','$1-'; $k.TrimEnd('-');"
@REM powershell -NoProfile -Command "$d=(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name DigitalProductId -ErrorAction Stop).DigitalProductId; $chars='BCDFGHJKMPQRTVWXY2346789'; $key=''; for($i=24;$i -ge 0;$i--){ $cur=0; for($j=14;$j -ge 0;$j--){ $cur = $cur*256 + $d[$j + $i*15] } for($k=24;$k -ge 0;$k--){ $key = $chars[$cur % 29] + $key; $cur = [math]::Floor($cur/29) } $key = $key.Substring(1) } $key = $key -replace '(.{5})','$1-'; $key.TrimEnd('-')"
powershell -NoProfile -Command "$d=(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name DigitalProductId -ErrorAction Stop).DigitalProductId; $chars='BCDFGHJKMPQRTVWXY2346789'; $key=''; for($i=24;$i -ge 0;$i--){ $cur=0; for($j=14;$j -ge 0;$j--){ $cur = $cur*256 + $d[$j + $i*15] } for($k=24;$k -ge 0;$k--){ $key = $chars[$cur % 29] + $key; $cur = [math]::Floor($cur/29) } $key = $key.Substring(1) } $key = $key -replace '(.{5})','$1-'; $out=$key.TrimEnd('-'); Write-Output $out; Write-Output ('Length: ' + $out.Length)"
```

- verify activation status

```bat
slmgr.vbs /dlv
```
