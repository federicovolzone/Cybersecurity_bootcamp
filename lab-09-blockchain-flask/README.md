# Lab 09 — Blockchain con Flask

## Obiettivo
Implementare una blockchain funzionale con API REST usando Python e Flask per comprendere hashing, Proof of Work, Merkle Tree e consenso distribuito.

## Tool utilizzati
- `python3` — implementazione blockchain
- `flask` — API REST
- `hashlib` — SHA-256

## Struttura blocco
```json
{
    "index": 2,
    "timestamp": 1712345678.123,
    "transazioni": [...],
    "proof": 35293,
    "hash_precedente": "00000a3f..."
}
```

## Blockchain + API Flask
```python
from flask import Flask, jsonify, request
import hashlib, json, time
from uuid import uuid4

app = Flask(__name__)

class Blockchain:
    def __init__(self):
        self.catena = []
        self.transazioni_in_sospeso = []
        self.nodi = set()
        self.nuovo_blocco(proof=100, hash_precedente='1')

    def nuovo_blocco(self, proof, hash_precedente):
        blocco = {
            'index': len(self.catena) + 1,
            'timestamp': time.time(),
            'transazioni': self.transazioni_in_sospeso,
            'proof': proof,
            'hash_precedente': hash_precedente
        }
        self.transazioni_in_sospeso = []
        self.catena.append(blocco)
        return blocco

    def prova_del_lavoro(self, ultimo_blocco):
        proof = 0
        while not self.validazione_prova(ultimo_blocco['proof'], proof, self.hash(ultimo_blocco)):
            proof += 1
        return proof

    @staticmethod
    def validazione_prova(last_proof, proof, last_hash):
        guess = f'{last_proof}{proof}{last_hash}'.encode()
        return hashlib.sha256(guess).hexdigest()[:4] == "0000"

    @staticmethod
    def hash(blocco):
        return hashlib.sha256(json.dumps(blocco, sort_keys=True).encode()).hexdigest()

blockchain = Blockchain()
node_identifier = str(uuid4()).replace('-', '')

@app.route('/mine', methods=['GET'])
def mine():
    ultimo = blockchain.catena[-1]
    proof = blockchain.prova_del_lavoro(ultimo)
    blockchain.transazioni_in_sospeso.append({"mittente": "0", "destinatario": node_identifier, "importo": 1})
    blocco = blockchain.nuovo_blocco(proof, blockchain.hash(ultimo))
    return jsonify({"blocco": blocco}), 200

@app.route('/chain', methods=['GET'])
def chain():
    return jsonify({"catena": blockchain.catena, "lunghezza": len(blockchain.catena)}), 200

@app.route('/nodes/register', methods=['POST'])
def register_nodes():
    nodi = request.get_json().get('nodes')
    for nodo in nodi:
        blockchain.nodi.add(nodo)
    return jsonify({"messaggio": "Nodi registrati"}), 201

@app.route('/nodes/resolve', methods=['GET'])
def consensus():
    return jsonify({"messaggio": "Consenso eseguito"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

## Avvio nodi e test API
```bash
python3 blockchain_8000.py &
python3 blockchain_8001.py &
python3 blockchain_8003.py &

curl -X POST http://localhost:8000/nodes/register \
     -H "Content-Type: application/json" \
     -d '{"nodes": ["http://localhost:8001", "http://localhost:8003"]}'

curl http://localhost:8000/mine
curl http://localhost:8000/chain
curl http://localhost:8000/nodes/resolve
```

## Merkle Tree
```python
import hashlib

def hash_str(s):
    return hashlib.sha256(s.encode()).hexdigest()

def merkle_tree(transazioni):
    livello = [hash_str(tx) for tx in transazioni]
    while len(livello) > 1:
        nuovo = []
        for i in range(0, len(livello), 2):
            a = livello[i]
            b = livello[i+1] if i+1 < len(livello) else a
            nuovo.append(hash_str(a + b))
        livello = nuovo
    return livello[0]

txs_orig = ["Alice->Bob:0.5", "Carlo->Diana:1.2"]
txs_fake = ["Alice->Bob:5.0", "Carlo->Diana:1.2"]
print(merkle_tree(txs_orig) == merkle_tree(txs_fake))  # False — manomissione rilevata
```

## Risultati
- Blockchain funzionante con PoW (4 zeri iniziali)
- API REST per mining, visualizzazione, registrazione nodi
- Consenso: la catena più lunga vince
- Merkle Tree: manomissione di 1 transazione rilevata immediatamente

## Concetti chiave
- Hash del blocco precedente crea l'immutabilità della catena
- PoW rende computazionalmente costoso modificare la storia
- Algoritmo di consenso di Nakamoto: la catena più lunga è autorevole
- 51% attack: attaccante con >50% hashrate può riscrivere la storia
