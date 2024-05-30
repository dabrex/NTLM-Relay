# NTLM-Relay Attack

## Threath Model: L'*attaccante* è un computer della rete del *target* (AiTM)

### NTLM Relay ###
NTLM Relay è un tipo di attacco informatico che sfrutta le debolezze del protocollo di autenticazione NTLM (NT LAN Manager) utilizzato principalmente nei sistemi Windows. In questo attacco, un *attaccante* intercetta le richieste di autenticazione NTLM tra un client e un server e le inoltra a un server di destinazione, ottenendo accesso non autorizzato.

### Infrastruttura creata per la simulazione ###
 - Windows 2019 Server - Domain Controller denominato "NTLM-DC"
 - Server (win 2019) denominato "NTLM-SRV01"
 - Macchina Vittima (win 2019) denominata "MTLMvictim"
 - Macchina Attaccante (Kali Linux) 

 #### Sintesi dell'attacco ####
 Preliminarmente si è effettuata una fase di "enumerazione" della rete *target* 
 Per rendere possibile l'attacco si è partiti da una mail di "spear phishing" contenente un link malevolo (T1566.002 Mitre Att@ck) in cui si informa il "destinatario" che è necessario effettuare il "download" di un aggiornamento "critico" del sistema operativo, da un sottodominio "microsoft.com". Per redirigere il link "malevolo" verso la macchina attaccante si è utilizzato un attacco "DNS Spoofing" (T1584.002 Att&ck). Per rendere "credibile" il sito da cui effettuare l'*aggiornamento* si è creata una fittizia pagina "microsoft" fornita da un server web in esecuzione sulla macchina dell'*attaccante* che riproduce il sito ufficiale e sulla quale è presente un pulsante "Download" che redirige ad una "risorsa di rete" sulla macchina dell'*attaccante*. Per carpire le credenziali utente dalla macchina *vittima* si è utilizzato sulla macchina *attaccante* il software "RESPONDER" che intercetta le richieste LLMNR (Link-Local Multicast Name Resolution) e NBT-NS(NetBIOS Name Service) che la macchina *vittima* utilizza quando non riesce a 'connettersi' ad un 'servizio' o un 'host' utililizzando DNS. "RESPONDER" rispode fingendosi l'host richiesto. La *vittima* invia le credenziali NTLM per autenticarsi. L'NTLMv2 intercettato può essere utilizzato:
 -  per ottenere le credenziali in 'chiaro' mediante 'password cracking offline" (T1110.002 Att&ck) [john_the_ripper, hashcat]
 - per attacchi 'pass_the_hash' (T1550.002 Att&ck). 
 
 Al fine di effettuare NTLM_Relay_Attack (T1557.001 Att&ck) si è utilizzato il software "NTLMRELAYX" che intercettando le credenziali della macchina *vittima* (NTLM-victim) carpite nel tentativo di "download dell'aggiornamento" (in realtà cerca di connettersi alla macchina *attaccante*),  le utilizza per autenticarsi sulla macchina *target* (NTLM-SRV01).   
 

**diobon**

```python
fun arg {

    codice;
}
```

[testo](https://)

----------

* efw
* eojfij