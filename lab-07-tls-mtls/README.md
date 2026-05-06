# Lab 07 — TLS/mTLS con X.509

## Obiettivo
Configurare comunicazioni cifrate con TLS e mutual TLS usando certificati X.509 per comprendere l'autenticazione bidirezionale.

## Tool utilizzati
- `openssl` — generazione certificati e test
- `python3` — server e client TLS/mTLS

## TLS Handshake
Client                              Server
|──── ClientHello ───────────────> |
| <── ServerHello ──────────────── |
| <── Certificate ──────────────── |
| <── [CertificateRequest] ──────  |  (solo mTLS)
| <── ServerHelloDone ──────────── |
|──── [Certificate] ─────────────> |  (solo mTLS)
|──── ClientKeyExchange ─────────> |
|──── Finished ──────────────────> |
| <── Finished ─────────────────── |
|       [dati cifrati] <->         |

## Generazione certificati con OpenSSL
```bash
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -out server.crt
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr
openssl x509 -req -days 3650 -in client.csr -CA ca.crt -CAkey ca.key -out client.crt
```

## Generazione certificati con Python
```python
from cryptography import x509
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import hashes, serialization
import datetime

ca_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
subject = issuer = x509.Name([x509.NameAttribute(x509.NameOID.COMMON_NAME, "MyCA")])
cert_ca = (x509.CertificateBuilder()
    .subject_name(subject)
    .issuer_name(issuer)
    .public_key(ca_key.public_key())
    .serial_number(x509.random_serial_number())
    .not_valid_before(datetime.datetime.utcnow())
    .not_valid_after(datetime.datetime.utcnow() + datetime.timedelta(days=3650))
    .add_extension(x509.BasicConstraints(ca=True, path_length=None), critical=True)
    .sign(ca_key, hashes.SHA256()))

# python3 server_client_cert_gen.py gen_certs
# python3 server_client_cert_gen.py server 0.0.0.0
# python3 server_client_cert_gen.py client 10.0.2.6
```

## Server mTLS Python
```python
import ssl, socket

context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
context.verify_mode = ssl.CERT_REQUIRED
context.load_cert_chain("server.crt", "server.key")
context.load_verify_locations(cafile="client.crt")
context.minimum_version = ssl.TLSVersion.TLSv1_2

with socket.socket() as sock:
    sock.bind(("0.0.0.0", 8080))
    sock.listen(5)
    conn, addr = sock.accept()
    with context.wrap_socket(conn, server_side=True) as ssock:
        print("TLS version:", ssock.version())
        data = ssock.recv(4096)
        ssock.sendall(data.upper())
```

## Client mTLS Python
```python
import ssl, socket

context = ssl.create_default_context(ssl.Purpose.SERVER_AUTH, cafile="server.crt")
context.load_cert_chain("client.crt", "client.key")
context.verify_mode = ssl.CERT_REQUIRED
context.check_hostname = False

with socket.create_connection(("10.0.2.6", 8080)) as raw:
    with context.wrap_socket(raw, server_hostname="10.0.2.6") as ssock:
        ssock.sendall(b"ciao dal client")
        print(ssock.recv(4096))
```

## Test connessione con OpenSSL
```bash
openssl s_client -connect 10.0.2.6:8080 -CAfile server.crt
openssl s_client -connect 10.0.2.6:8080 -CAfile server.crt -cert client.crt -key client.key
```

## Differenza TLS vs mTLS
| | TLS | mTLS |
|---|---|---|
| Autentica server | SI | SI |
| Autentica client | NO | SI |
| Uso tipico | Browser web | Microservizi, API interne, VPN |

## Risultati
- TLS funzionante con verifica certificato server
- mTLS funzionante con autenticazione bidirezionale
- Chain of trust X.509 verificata correttamente

## Concetti chiave
- TLS standard autentica solo il server — il lucchetto nel browser
- mTLS autentica entrambi — usato in architetture zero-trust
- I certificati X.509 sono la base dell'infrastruttura PKI
- Difesa attacchi: HSTS, certificate pinning, revoca certificati
