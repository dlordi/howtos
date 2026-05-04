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

# read DigitalProductId
$dpid = (Get-ItemProperty -Path $regPath -Name $valueName -ErrorAction Stop).$valueName

# convert DigitalProductId to Product Key
function Convert-DPIDToProductKey {
    param([byte[]]$dpid)
    $keyOffset = 52
    $isWin8 = ($dpid[66] -band 0xFF) / 6 -band 1

    if ($isWin8 -eq 1) {
        $key = ""
        $chars = "BCDFGHJKMPQRTVWXY2346789"
        $last = 0
        $dpid[66] = ($dpid[66] -band 0xF7) -bor (($last -shr 0) -band 0x08)
        for ($i = 24; $i -ge 0; $i--) {
            $cur = 0
            for ($j = 14; $j -ge 0; $j--) {
                $cur = $cur * 256 -bxor $dpid[$j + $keyOffset]
                $dpid[$j + $keyOffset] = [math]::Floor($cur / 24)
                $cur = $cur % 24
            }
            $key += $chars[$cur]
        }
        $key = $key.Substring(1, $key.Length -1) + "N" + $key[0]
        for ($k = 5; $k -lt $key.Length; $k += 6) {
            $key = $key.Insert($k, "-")
            $k++
        }
        return $key
    } else {
        $chars = "BCDFGHJKMPQRTVWXY2346789"
        $key = ""
        for ($i = 24; $i -ge 0; $i--) {
            $current = 0
            for ($j = 14; $j -ge 0; $j--) {
                $current = $current * 256 + $dpid[$j + $keyOffset]
                $dpid[$j + $keyOffset] = [math]::Floor($current / 24)
                $current = $current % 24
            }
            $key += $chars[$current]
        }
        $key = $key.ToCharArray() -join ""
        $result = ""
        for ($i = 0; $i -lt $key.Length; $i++) {
            if (($i % 5) -eq 0 -and $i -ne 0) { $result += "-" }
            $result += $key[$i]
        }
        return $result
    }
}

$dpidCopy = New-Object byte[] $dpid.Length
[Array]::Copy($dpid, $dpidCopy, $dpid.Length)

# show Product Key
Convert-DPIDToProductKey -dpid $dpidCopy
```

- verify activation status

```bat
slmgr.vbs /dlv
```
