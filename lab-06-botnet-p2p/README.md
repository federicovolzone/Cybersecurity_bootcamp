# Lab 06 — Botnet P2P

## Disclaimer
Progetto esclusivamente educativo in ambiente isolato. Uso al di fuori del laboratorio è illegale.

## Obiettivo
Implementare una botnet peer-to-peer per comprendere la struttura decentralizzata usata dai malware moderni per resistere al takedown.

## Ambiente
| VM | IP | Ruolo |
|---|---|---|
| Kali Linux | 10.0.2.6 | Nodo C2 |
| Lubuntu | 10.0.2.15 | Bot |

## Confronto C&C centralizzato vs P2P
| Caratteristica | Botnet C&C | Botnet P2P |
|---|---|---|
| Single point of failure | Sì | No |
| Resilienza | Bassa | Alta |
| Anonimato attaccante | Basso | Alto |
| Esempio reale | Mirai | Zeus P2P, GameOver Zeus |

## Script — p2p_node_v5.py
```python
import socket, threading

class P2PNode:
    def __init__(self, listen_port):
        self.listen_port = listen_port
        self.peers = {}
        self.peers_lock = threading.Lock()
        self.running = True

    def start(self):
        self.server_socket = socket.socket()
        self.server_socket.bind(("0.0.0.0", self.listen_port))
        self.server_socket.listen(5)
        threading.Thread(target=self._listen_loop, daemon=True).start()
        print(f"[*] Nodo P2P in ascolto su porta {self.listen_port}")

    def _listen_loop(self):
        while self.running:
            try:
                client_sock, addr = self.server_socket.accept()
                with self.peers_lock:
                    self.peers[addr] = client_sock
                threading.Thread(
                    target=self._handle_peer,
                    args=(client_sock, addr),
                    daemon=True
                ).start()
            except:
                break

    def _handle_peer(self, sock, addr):
        try:
            while True:
                data = sock.recv(1024)
                if not data:
                    break
                print(f"[{addr[0]}:{addr[1]}] {data.decode()}")
        finally:
            with self.peers_lock:
                self.peers.pop(addr, None)

    def connect_to(self, ip, port):
        sock = socket.socket()
        sock.connect((ip, port))
        with self.peers_lock:
            self.peers[(ip, port)] = sock
        print(f"[+] Connesso a {ip}:{port}")

    def broadcast(self, message):
        with self.peers_lock:
            for addr, sock in list(self.peers.items()):
                try:
                    sock.sendall(message.encode())
                except:
                    self.peers.pop(addr, None)
```

## Esecuzione
```bash
# Su Kali (porta 5000)
python3 p2p_node.py 5000

# Su Lubuntu (porta 5001)
python3 p2p_node.py 5001
```

## Risultati
- Rete P2P funzionante con 2 nodi
- Comandi propagati tra i bot senza server centrale
- Comprensione della resilienza delle botnet decentralizzate

## Concetti chiave
- Botnet P2P non hanno single point of failure
- Comunicazione su porte comuni (80, 443) per bypassare firewall
- Difesa: network segmentation, traffic analysis, threat intelligence
