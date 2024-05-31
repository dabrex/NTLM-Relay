# NTLM-Relay Attack

## Threath Model: L'*attaccante* è un computer della rete del *target* (AiTM)

### NTLM Relay ###
NTLM Relay è un tipo di attacco informatico che sfrutta le debolezze del protocollo di autenticazione NTLM (NT LAN Manager) utilizzato principalmente nei sistemi Windows. In questo attacco, un *attaccante* intercetta le richieste di autenticazione NTLM tra un client e un server e le inoltra a un server di destinazione, ottenendo accesso non autorizzato.

### Infrastruttura creata per la simulazione ###
 - Windows 2019 Server - Domain Controller denominato "NTLM-DC"
 - Server (win 2019) denominato "NTLM-SRV01"
 - Macchina Vittima (win 2019) denominata "MTLMvictim"
 - Macchina Attaccante (Kali Linux) 

 ### Sintesi dell'attacco ###
 Preliminarmente si è effettuata una fase di "enumerazione" della rete *target* (T1046 Mitre Att&ck)............... attraverso i software "NMAP", "CRACKMAPEXEC", "NBTSCAN".
 Per rendere possibile l'attacco si è partiti da una mail di "spear phishing" contenente un link malevolo (T1566.002  Att@ck) in cui si informa la "vittima" che è necessario effettuare il "download" di un aggiornamento "critico" del sistema operativo, da un sottodominio "microsoft.com". Per redirigere il link "malevolo" verso la macchina attaccante si è utilizzato un attacco "DNS Spoofing" (T1584.002 Att&ck) [Software ETTERCAP]. Per rendere "credibile" il sito da cui effettuare l'*aggiornamento* si è creata una fittizia pagina "microsoft" fornita da un server web in esecuzione sulla macchina dell'*attaccante* che riproduce il sito ufficiale e sulla quale è presente un pulsante "Download" che redirige alla macchina dell'*attaccante*. Per carpire le credenziali utente dalla macchina *vittima* si è utilizzato sulla macchina *attaccante* il software "RESPONDER" che intercetta le richieste LLMNR (Link-Local Multicast Name Resolution) e NBT-NS(NetBIOS Name Service) che la macchina *vittima* utilizza quando non riesce a 'connettersi' ad un 'servizio' o un 'host' utililizzando DNS. "RESPONDER" risponde fingendosi l'host richiesto. La *vittima* invia le credenziali NTLM per autenticarsi. L'NTLMv2 intercettato può essere utilizzato:
 -  per ottenere le credenziali in 'chiaro' mediante 'password cracking offline" (T1110.002 Att&ck) [john_the_ripper, hashcat]
 - per attacchi 'pass_the_hash' (T1550.002 Att&ck). 
 
 Al fine di effettuare NTLM_Relay_Attack (T1557.001 Att&ck) si è utilizzato il software "NTLMRELAYX" che intercettando le credenziali della macchina *vittima* (NTLM-victim) carpite nel tentativo di "download dell'aggiornamento" (in realtà la *vittima* cerca di connettersi alla macchina *attaccante*),  le utilizza per autenticarsi sulla macchina *target* (NTLM-SRV01). Tale operazione viene effettuata attivando l'opzione che avvia un server "SOCKS" locale che permette all'*attaccante* di eseguire ulteriori operazioni di rete sulla macchina *target*.

 Una volta acceduti, per garantire la "persistenza"  si è creato un file eseguibile windows che avvia una "reverse shell" (T1219 Att&ck) già 'settata' per connettersi all'indirizzo IP dell'*attaccante* ad una determinata porta. A supporto si è utilizzato il software "MSFVENOM". 

 Per il trasferimento della 'reverse-shell' sul *target* (NTLM-SRV01) si è utilizzato il software "PROXYCHAIN4" che sfruttando il server 'socks' aperto con 'ntlmrelayx' permette di eseguire il software "SMBCLIENT" utilizzato per interagire con condivisioni di rete "SMB" sul *target*.

 La "reverse-shell" è stata posizionata nella cartella "C:\Windows\System32" sfruttando il fatto che le credenziali utilizzate nel relay hanno 'diritti amministrativi' ed il file è stato denominato 'whoani.exe' nel tentativo di 'confonderlo' con il già presente "whoami.exe". 

 Sulla macchina *attaccante* si è avviato il software "NC (netcat)" settandolo in ascolto sulla porta impostata nella "reverse-shell". 

 Per 'avviare' il processo della 'reverse-shell' si è sfruttato ancora 'proxychain4' per eseguire il software "SMBEXEC".  

 Al fine di rendere 'definitiva' la persistenza nel sistema *target* si è impostata una "Scheduled Task" (T1053.005 Att&ck) che in fase di avvio del sistema *target* mette in esecuzione la 'reverse-shell'. 

 Per eliminare le 'tracce' dell'attività eseguita sul *target* si è utilizzato un comando "POWERSHELL" impartito attraverso 'smbexec' che rimuove tutti i "log di sistema" sul *target*.

 -------------------

 ### Dettaglio dell'attacco passo-passo ###

#### Passo 1 - Enumerazione NMAP ####

```python
 sudo nmap -sV 10.0.0.0/24 //(identificazione servizi in esecuzione sulla rete)
 ```
Da questa prima fase di 'enumerazione' si recuperano le seguenti informazioni:
- nr.03 macchine windows con indirizzi **IP 10.0.0.1, 10.0.0.10, 10.0.0.100**
- macchina 10.0.0.100 = Domain controller; nome macchina = **NTLM-DC**; nome dominio = **NTLMLAB.local**; ha il servizio **DNS** attivo.

#### Passo 2 - Enumerazione CRACKMAPEXEC ####

 ```python
 sudo crackmapexec smb 10.0.0.0/24 --gen-relay-list /home/kali/Desktop/targets2.txt  //(identificazione servizi SMB esecuzione sulla rete e generazione di lista in file .txt)
 ```

Da questa fase di 'enumerazione' si recuperano le seguenti informazioni:
- nr.02 macchine windows con SMB attivo utilizzabile per NTLM Relay con indirizzi **IP 10.0.0.1, 10.0.0.10**

#### Passo 3 - Enumerazione NBTSCAN ####

 ```python
 sudo nbtscan 10.0.0.0/24  //(identificazione nomi 'netbios' delle macchine sulla rete)
 ```

Da questa fase di 'enumerazione' si recuperano le seguenti informazioni:
- nr.03 macchine windows con nomi macchina **VICTIM, SRV01, NTLM-DC**

#### Passo 4 - Preparazione Finta Pagina Microsoft ####
- Creazione di Finta Pagina HTML con CSS e immagini
- Caricamento del materiale HTML 'prodotto' nella cartella '/var/www/html/' sulla macchina *attaccante*
- Avvio del server web 'Apache' sulla macchina *attaccante*
- Risultato della 'Finta Pagina Microsoft' visualizzabile [qui](https://github.com/dabrex/NTLM-Relay/main/Sito-Microsoft-Fasullo.jpg)

[testo](https://)
1. first
2. second
3. third
