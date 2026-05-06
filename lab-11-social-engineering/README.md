# Lab 11 — Social Engineering

## Obiettivo
Analizzare le tecniche di manipolazione psicologica usate negli attacchi di social engineering e studiare i metodi OSINT per la raccolta di informazioni sui target.

## Tool utilizzati
- `theHarvester` — raccolta email, subdomini, host
- `nmap` — scansione target
- `whois`, `dig`, `nslookup` — OSINT su domini

## Tecniche di Social Engineering
| Tecnica | Descrizione | Esempio |
|---|---|---|
| Phishing | Email falsa che imita azienda legittima | "Rinnova la tua password bancaria" |
| Spear Phishing | Phishing mirato a persona specifica | Email personalizzata con nome e ruolo |
| Pretexting | Scenario inventato per guadagnare fiducia | "Sono dell'IT, ho bisogno delle tue credenziali" |
| Baiting | Esche fisiche o digitali | USB con etichetta "Stipendi 2024" |
| Tailgating | Accesso fisico non autorizzato | Entrare in ufficio dietro un dipendente |
| Vishing | Phishing via telefono | "Sono di Microsoft Support..." |

## Principi psicologici sfruttati
1. **Autorità** — fingersi figure di potere (IT, management, banca)
2. **Urgenza** — creare pressione temporale per bypassare il pensiero critico
3. **Reciprocità** — offrire qualcosa per ottenere qualcosa in cambio
4. **Social proof** — "tutti gli altri colleghi l'hanno già fatto"
5. **Fiducia** — costruire rapporto prima di chiedere informazioni

## OSINT — Raccolta informazioni
```bash
whois target.com
nslookup target.com
dig target.com ANY

theHarvester -d target.com -b google,linkedin,shodan

# Google Dorks
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com ext:sql
"@target.com" "password"
```

## Scansione target
```bash
nmap -sn 10.0.2.0/24
nmap -sV -O -p- 10.0.2.5
nmap --script vuln 10.0.2.5
nmap -sV 10.0.2.5 -oN scan_risultati.txt
```

## Kill Chain — Social Engineering

Raccolta informazioni (OSINT)
|
Identificazione del target
|
Costruzione del pretesto
|
Esecuzione dell'attacco
|
Estrazione credenziali/accesso
|
Post-exploitation


## Risultati
- Comprensione dei vettori di attacco non tecnici
- OSINT completato: email, subdomini, host attivi identificati
- Capacità di riconoscere tentativi di manipolazione

## Concetti chiave
- Il fattore umano è spesso l'anello più debole della sicurezza
- Un firewall non può bloccare un dipendente che apre un allegato email
- Difesa principale: security awareness training continuo
- Verifica out-of-band prima di fornire qualsiasi informazione sensibile
