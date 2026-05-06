Lab-08! Clicca "+" → Create new file, scrivi:
lab-08-diffie-hellman/README.md
Incolla questo:
markdown# Lab 08 — Diffie-Hellman Key Exchange

## Obiettivo
Implementare lo scambio di chiavi Diffie-Hellman per comprendere come due parti concordano una chiave segreta su un canale pubblico senza mai trasmetterla.

## Tool utilizzati
- `python3` — implementazione con libreria `cryptography`

## Schema matematico
Parametri pubblici condivisi: p (primo grande), g (generatore)
Alice: sceglie privato a  -->  pubblica A = g^a mod p
Bob:   sceglie privato b  -->  pubblica B = g^b mod p
Alice calcola: S = B^a mod p
Bob   calcola: S = A^b mod p
--> S identico per entrambi — senza mai scambiare a o b

## Script — scambio_criptato_chiavi.py
```python
from cryptography.hazmat.primitives.asymmetric.dh import generate_parameters
from cryptography.hazmat.backends import default_backend

# 1. Parametri DH 2048-bit condivisi pubblicamente
print("[*] Generazione parametri DH 2048-bit...")
parameters = generate_parameters(generator=2, key_size=2048, backend=default_backend())

# 2. Alice e Bob generano le loro coppie di chiavi
alice_private = parameters.generate_private_key()
bob_private   = parameters.generate_private_key()

# 3. Scambio chiavi pubbliche (su canale insicuro)
alice_public = alice_private.public_key()
bob_public   = bob_private.public_key()

# 4. Ognuno calcola il segreto condiviso
alice_secret = alice_private.exchange(bob_public)
bob_secret   = bob_private.exchange(alice_public)

# 5. Verifica — devono essere identici
assert alice_secret == bob_secret, "ERRORE: segreti diversi!"
print(f"[+] Segreto condiviso: {alice_secret.hex()[:32]}...")
print("[+] Alice e Bob hanno la stessa chiave senza averla trasmessa")

# Salva su file per verifica
with open("AliceSharedSecret.bin", "wb") as f:
    f.write(alice_secret)
with open("BobSharedSecret.bin", "wb") as f:
    f.write(bob_secret)
```

## Esecuzione
```bash
pip install cryptography
python3 scambio_criptato_chiavi.py

# Verifica che i segreti siano identici
diff AliceSharedSecret.bin BobSharedSecret.bin && echo "IDENTICI" || echo "DIVERSI"
```

## File generati
| File | Contenuto |
|---|---|
| AliceSharedSecret.bin | Segreto DH derivato lato Alice |
| BobSharedSecret.bin | Segreto DH derivato lato Bob |
| parametersPG.pem | Parametri DH pubblici condivisi |

## Risultati
- Chiave condivisa identica su entrambi i lati (assert passato)
- La chiave privata non è mai stata trasmessa
- Segreto di 256 byte generato da parametri 2048-bit

## Concetti chiave
- DH vulnerabile a MitM senza autenticazione — serve combinarlo con X.509
- ECDH è la versione moderna più efficiente — stessa sicurezza con chiavi più corte
- È la base del Perfect Forward Secrecy in TLS
- Ogni sessione usa chiavi DH diverse — sessioni passate rimangono sicure anche se la chiave privata viene compromessa
