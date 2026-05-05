# Lab 01 — MAC/ARP Spoofing

## Obiettivo
Intercettare il traffico di rete tra due macchine in una rete locale simulando un attacco Man-in-the-Middle tramite ARP spoofing.

## Ambiente
- **Attaccante:** Kali Linux (10.0.2.6)
- **Vittima:** Lubuntu (10.0.2.15)
- **Gateway:** pfSense (10.0.2.1)
- **Rete:** VirtualBox NatNetwork 10.0.2.0/24

## Tool utilizzati
- `bettercap` — ARP spoofing e intercettazione traffico
- `wireshark` — analisi e sniffing dei pacchetti
- `arp` — verifica delle tabelle ARP prima/dopo l'attacco

## Procedura

### 1. Verifica della rete
Scansione della rete per identificare gli host attivi e i relativi MAC address.

### 2. ARP Spoofing con Bettercap
Avvio di Bettercap e configurazione del modulo ARP per avvelenare le tabelle ARP della vittima e del gateway, posizionando Kali come man-in-the-middle.

### 3. Sniffing del traffico
Con Wireshark in ascolto sull'interfaccia di rete, cattura del traffico intercettato tra vittima e gateway — incluse credenziali in chiaro su protocolli non cifrati.

### 4. Verifica dell'attacco
Controllo delle tabelle ARP sulla macchina vittima per confermare che il MAC del gateway fosse stato sostituito con quello di Kali.

## Risultati
- Traffico HTTP intercettato con successo
- Credenziali in chiaro recuperate dallo sniffing
- Tabelle ARP avvelenate correttamente su entrambe le macchine

## Concetti chiave
- Il protocollo ARP non prevede autenticazione — è intrinsecamente vulnerabile
- Gli attacchi MitM sono la base di molte tecniche offensive più avanzate
- Difesa: ARP dinamico sicuro, VLAN, 802.1X
