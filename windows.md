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
    $isWin8 = (($dpid[66] -band 0xFF) -shr 3) -band 1

    $chars = "BCDFGHJKMPQRTVWXY2346789"
    if ($isWin8 -eq 1) {
        $dp = New-Object byte[] 15
        [Array]::Copy($dpid, $keyOffset, $dp, 0, 15)

        $last = 0
        for ($i = 0; $i -lt 15; $i++) { $last = ($last -bxor $dp[$i]) }
        $dp[14] = ($dp[14] -bxor ($last -shr 0))

        $key = ""
        for ($i = 24; $i -ge 0; $i--) {
            $acc = 0
            for ($j = 14; $j -ge 0; $j--) {
                $acc = $acc * 256 -bxor $dp[$j]
                $dp[$j] = [math]::Floor($acc / 24)
                $acc = $acc % 24
            }
            $key += $chars[$acc]
        }
        $key = $key.Substring(1) + "N" + $key[0]
    } else {
        $dp = New-Object byte[] 15
        [Array]::Copy($dpid, $keyOffset, $dp, 0, 15)

        $key = ""
        for ($i = 24; $i -ge 0; $i--) {
            $acc = 0
            for ($j = 14; $j -ge 0; $j--) {
                $acc = $acc * 256 + $dp[$j]
                $dp[$j] = [math]::Floor($acc / 24)
                $acc = $acc % 24
            }
            $key += $chars[$acc]
        }
        $key = $key.ToCharArray() -join ""
    }

    $out = ""
    for ($i = 0; $i -lt $key.Length; $i++) {
        if (($i -ne 0) -and (($i % 5) -eq 0)) { $out += "-" }
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
