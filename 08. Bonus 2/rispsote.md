# Risposte alle domande

### Che tipo di transazioni si sono verificate tra il client e il server in questo attacco?

la sequenza delle azioni eseguite (SRC: 209.165.201.17) sul sistema bersaglio (DST: 209.165.200.235):

    Accesso e verifica permessi:
        Esegue id e il server risponde con uid=0(root) gid=0(root), confermando che l'attaccante ha già i privilegi di root (amministratore).

        Esegue whoami, hostname (che restituisce metasploitable) e ifconfig per raccogliere informazioni sulla rete e sull'host.

    Lettura delle credenziali riservate:
        Esegue cat /etc/shadow per visualizzare gli hash delle password di tutti gli utenti del sistema.

    Creazione di una backdoor (Persistenza):
        Inserisce un nuovo utente amministratore (myroot) nel file /etc/shadow usando il comando:
        echo "myroot::14747:0:99999:7:::" >> /etc/shadow

        Legge il file degli utenti con cat /etc/passwd.
        Aggiunge l'utente myroot con privilegi root (UID 0) nel file degli utenti:
        echo "myroot:x:0:0:root:/root:/bin/bash" >> /etc/passwd

        Verifica la creazione con grep root /etc/passwd e infine esce dalla sessione con exit.

In sintesi, l'attaccante ha sfruttato la sessione root per rubare le password e creare un nuovo utente amministratore abusivo (myroot) per garantirsi l'accesso futuro anche se la vulnerabilità iniziale venisse corretta.

### Cosa hai osservato? 

Si osserva l'intera conversazione TCP riassemblata in chiaro tra l'attaccante e il server. L'attaccante ha ottenuto una shell remota interattiva sul bersaglio ed sta eseguendo comandi di sistema (come id, whoami, hostname, ifconfig) ottenendo risposte dirette con privilegi da amministratore (root).

### Cosa indicano i colori del testo rosso e blu?

- Testo Rosso: Rappresenta il traffico inviato dal client / attaccante (209.165.201.17) verso il server (i comandi digitati).
- Testo Blu: Rappresenta la risposta inviata dal server / bersaglio (209.165.200.235) verso il client (l'output dei comandi eseguiti).


### Cosa rivela questo sul ruolo dell'attaccante sul computer bersaglio?

Rivela che l'attaccante ha ottenuto il controllo completo e amministrativo (accesso ROOT) sul computer bersaglio.

- L'output del comando whoami restituisce root.
- L'output del comando id restituisce uid=0(root) gid=0(root).
- L'attaccante non è un utente standard, ma ha il livello massimo di privilegi, il che gli permette di leggere qualsiasi file, modificare la configurazione di sistema e creare nuovi utenti a suo piacimento.

### Scorri il flusso TCP. Che tipo di dati ha letto l'attore della minaccia?

Scorrendo il flusso TCP, si vede che l'attaccante ha letto ed eseguito l'exfiltration dei seguenti file di sistema altamente sensibili:

- /etc/shadow (tramite il comando cat /etc/shadow): Contiene gli hash delle password di tutti gli utenti di sistema (inclusi root, msfadmin, postgres, analyst, ecc.). Con questi dati, l'attaccante può tentare di decifrare le password off-line per riutilizzarle su altri sistemi.
- /etc/passwd (tramite i comandi cat /etc/passwd e grep root /etc/passwd): Contiene l'elenco completo degli account utente del sistema, le loro directory home e le shell di login assegnate.

### Quali sono gli indirizzi IP e i numeri di porta di origine e destinazione per il traffico FTP?

- SRC_IP 192.168.0.11:52776 
- DEST_IP 209.165.200.235:21

### Quali sono le credenziali utente per accedere al sito FTP?

user: analyst
pass: cyberops


### Qual è il contenuto del file? Ricorda che uno dei servizi elencati nel grafico a torta è ftp_data.

Il contenuto del file confidential.txt è:

CONFIDENTIAL DOCUMENT
DO NOT SHARE
This document contains information about the last securityt breach

### Quali sono i diversi tipi di file? Guarda la sezione MIME Type dello schermo. 

- text/plain
- image/jpg
- image/png
- text/html
- image/gif
- image/x-icon

### Scorri fino all'intestazione Files - Source. Quali sono le sorgenti dei file elencate?

- HTTP 22
- FTP_DATA 1


### Qual è il tipo MIME, l'indirizzo IP di origine e di destinazione associato al trasferimento dei dati FTP? 

- text/plain
- SRC_IP 192.168.0.11
- DEST_IP 209.165.200.235

### Quando si è verificato questo trasferimento?

june 11th 2020, 03:53:09.088

### Qual è il contenuto testuale del file trasferito tramite FTP?

CONFIDENTIAL DOCUMENT
DO NOT SHARE
This document contains information about the last securityt breach

### Con tutte le informazioni raccolte finora, qual è la tua raccomandazione per fermare ulteriori accessi non autorizzati?

Sulla base dell'analisi svolta nei log, l'attaccante ha ottenuto l'accesso root, creato la backdoor myroot nei file /etc/passwd e /etc/shadow, ed esfiltrato dati tramite FTP.

Per neutralizzare la minaccia e fermare ulteriori accessi non autorizzati, ecco le azioni immediate e a lungo termine da applicare:

Contenimento Immediato

    Isolare l'host compromesso: Staccare immediatamente la macchina 209.165.200.235 dalla rete per impedire ulteriori esfiltrazioni o movimenti laterali.
    Rimuovere la backdoor: Eliminare l'account abusivo myroot creato dall'attaccante sia da /etc/passwd che da /etc/shadow.
    Invalidate le credenziali: Forzare il cambio password immediato per tutti gli account di sistema (in particolare root, analyst e gli utenti FTP come admin).
    Terminare le sessioni attive: Identificare e chiudere qualsiasi processo o shell remota attiva (come il demone avviato sulla porta non standard 6200).

Eradicazione e Remediation

    Applicare le patch di sicurezza: Identificare la vulnerabilità del servizio che ha permesso la shell remota iniziale (es. servizio vulnerabile su porta 6200/Metasploitable) e aggiornare o disabilitare il servizio.
    Bloccare l'IP dell'attaccante: Inserire gli indirizzi IP sorgente dell'attacco (209.165.201.17 e 192.168.0.11) nelle regole del firewall / NIDS per bloccarne il traffico.
    Disabilitare o proteggere FTP: Sostituire il protocollo FTP in chiaro con alternative cifrate come SFTP/SCP e disabilitare l'accesso FTP anonimo o con credenziali di default.