# Lab 04 — Caesar Cipher & RSA

## Obiettivo
Implementare e confrontare crittografia simmetrica classica (Cesare) e asimmetrica moderna (RSA-OAEP).

## Tool utilizzati
- `python3` — implementazione algoritmi
- `openssl` — operazioni RSA

## Script — cifrario di Cesare
```python
import string

ALFABETO_ESTESO = string.ascii_lowercase + string.digits + " .,!?;:@"

def cifrario_ottimizzato(testo, chiave, modalita):
    n = len(ALFABETO_ESTESO)
    shift = chiave % n
    if modalita == "decript":
        shift = -shift
    mappa = {char: i for i, char in enumerate(ALFABETO_ESTESO)}
    risultato = []
    for char in testo:
        lower = char.lower()
        if lower in mappa:
            idx = (mappa[lower] + shift) % n
            nuovo = ALFABETO_ESTESO[idx]
            risultato.append(nuovo.upper() if char.isupper() else nuovo)
        else:
            risultato.append(char)
    return "".join(risultato)

testo = "Ciao Mondo"
chiave = 5
cifrato = cifrario_ottimizzato(testo, chiave, "cript")
decifrato = cifrario_ottimizzato(cifrato, chiave, "decript")
print(f"Originale: {testo}")
print(f"Cifrato:   {cifrato}")
print(f"Decifrato: {decifrato}")

# Brute force
for k in range(len(ALFABETO_ESTESO)):
    print(f"Chiave {k:3d}: {cifrario_ottimizzato(cifrato, k, 'decript')}")
```

## Comandi OpenSSL — RSA
```bash
openssl genrsa -out pub_priv_pair.key 2048
openssl rsa -in pub_priv_pair.key -pubout -out public.key
openssl pkeyutl -encrypt -inkey public.key -pubin \
        -in plaintext.txt -out cipher.bin \
        -pkeyopt rsa_padding_mode:oaep
openssl pkeyutl -decrypt -inkey pub_priv_pair.key \
        -in cipher.bin -out plainD.txt \
        -pkeyopt rsa_padding_mode:oaep
cat plainD.txt
```

## Confronto Caesar vs RSA
| Caratteristica | Caesar | RSA |
|---|---|---|
| Tipo | Simmetrica | Asimmetrica |
| Chiavi | 1 condivisa | Pubblica + Privata |
| Sicurezza | Debole | Forte (2048 bit) |
| Brute force | Secondi | Computazionalmente impossibile |
| Padding | No | OAEP obbligatorio |

## Risultati
- Caesar rotto via brute force in meno di 1 secondo
- RSA con OAEP funzionante con chiavi generate dinamicamente

## Concetti chiave
- Caesar è vulnerabile al brute force — solo 25 chiavi possibili
- RSA si basa sulla difficoltà di fattorizzare grandi numeri primi
- Il padding OAEP è fondamentale — senza è vulnerabile ad attacchi statistici
- La crittografia asimmetrica risolve il problema dello scambio di chiavi
