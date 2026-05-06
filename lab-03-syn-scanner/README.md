
Eccolo:
markdown# Lab 03 — SYN Scanner Custom

## Obiettivo
Sviluppare uno scanner di porte TCP con Scapy che esegue half-open scan senza completare il three-way handshake.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Scanner |
| Metasploitable2 | 10.0.2.5 | Target |

## Tool utilizzati
- `scapy` — costruzione pacchetti TCP SYN
- `python3` — script dello scanner

## Script — syn_scan.py
```python
from scapy.all import IP, TCP, sr1, sr, conf

conf.verb = 0

def syn_scan(ip, port):
    syn_packet = IP(dst=ip) / TCP(dport=port, flags='S')
    resp = sr1(syn_packet, timeout=1, verbose=0)
    if resp is None:
        return "filtered"
    if resp.haslayer(TCP):
        flags = resp[TCP].flags
        if flags == 0x12:
            rst = IP(dst=ip)/TCP(dport=port, flags='R', seq=int(resp[TCP].ack))
            sr1(rst, timeout=1, verbose=0)
            return "open"
        elif flags & 0x04:
            return "closed"
    return "filtered"

ip_target = "10.0.2.5"
packets = [IP(dst=ip_target)/TCP(dport=p, flags='S') for p in range(1, 1025)]
answered, _ = sr(packets, timeout=2, verbose=0, inter=0.001)
for snd, rcv in answered:
    if rcv.haslayer(TCP) and rcv[TCP].flags == 0x12:
        print(f"[OPEN] Porta {snd[TCP].dport}")
```

## Esecuzione
```bash
sudo python3 syn_scan.py
sudo nmap -sS 10.0.2.5 -p 1-1024
```

## Risultati
- Porte aperte identificate su Metasploitable2
- Nessun handshake completato — stealth scan

## Concetti chiave
- SYN scan non completa il handshake — meno visibile nei log applicativi
- Richiede privilegi root per inviare raw packets
- È la base del flag -sS di Nmap
- Difesa: IDS/IPS, rate limiting sulle connessioni
