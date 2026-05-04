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
powershell -NoProfile -Command "$d=(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -Name DigitalProductId -ErrorAction Stop).DigitalProductId; $chars='BCDFGHJKMPQRTVWXY2346789'; $key=''; for($i=24;$i -ge 0;$i--){ $cur=0; for($j=14;$j -ge 0;$j--){ $cur = $cur*256 + $d[$j + $i*15] } for($k=24;$k -ge 0;$k--){ $key = $chars[$cur % 29] + $key; $cur = [math]::Floor($cur/29) } $key = $key.Substring(1) } $key = $key -replace '(.{5})','$1-'; $key.TrimEnd('-')"
```

```ps1
$regPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
$valueName = "DigitalProductId"
$dpid = (Get-ItemProperty -Path $regPath -Name $valueName -ErrorAction Stop).$valueName

function Convert-DPIDToProductKey {
    param([byte[]]$dpid)
    $keyOffset = 52
    $chars = "BCDFGHJKMPQRTVWXY2346789"

    $dp = New-Object byte[] 15
    [Array]::Copy($dpid, $keyOffset, $dp, 0, 15)

    $keyChars = New-Object System.Text.StringBuilder
    for ($i = 24; $i -ge 0; $i--) {
        $acc = 0
        for ($j = 14; $j -ge 0; $j--) {
            $acc = ($acc * 256) + $dp[$j]
            $dp[$j] = [math]::Floor($acc / 24)
            $acc = $acc % 24
        }
        $keyChars.Append($chars[$acc]) | Out-Null
    }

    $key = $keyChars.ToString()

    if ($key.Length -lt 25) { $key = $key.PadLeft(25, 'B') }
    if ($key.Length -gt 25) { $key = $key.Substring(0,25) }

    $out = ""
    for ($i = 0; $i -lt 25; $i++) {
        if ($i -ne 0 -and ($i % 5) -eq 0) { $out += "-" }
        $out += $key[$i]
    }
    return $out
}

Convert-DPIDToProductKey -dpid $dpid
```

- verify activation status

```bat
slmgr.vbs /dlv
```
