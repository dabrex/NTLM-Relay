# NTLM-Relay Attack

## Threath Model: L'*attaccante* è un computer della rete del *target* (AiTM)

### NTLM Relay ###
NTLM Relay è un tipo di attacco informatico che sfrutta le debolezze del protocollo di autenticazione NTLM (NT LAN Manager) utilizzato principalmente nei sistemi Windows. In questo attacco, un *attaccante* intercetta le richieste di autenticazione NTLM tra un client e un server e le inoltra a un server di destinazione, ottenendo accesso non autorizzato.

### Infrastruttura creata per la simulazione ###
 - Windows 2019 Server - Domain Controller denominato "AD...."
 - Server (win 2019) denominato "Server 01"
 - Macchina Vittima (win 2019) denominata "victim"
 - Macchina Attaccante (Kali Linux) 

 #### Sintesi dell'attacco ####
 Per rendere possibile l'attacco si è partiti da una mail di "spear phishing" contenente un un link malevolo (T1566.002 Mitre Att@ck) in cui si informa il "destinatario" che è necessario effettuare il "download" di un aggiornamento "critico" del sistema operativo, da un sottodominio "microsoft.com". Per redirigere il link "malevolo" verso la macchina attaccante si è utilizzato un attacco "DNS Spoofing" (T1584.002 Atta&ck). Per rendere "credibile" il sito da cui effettuare l'*aggiornamento* si è creata una fittizia pagina "microsoft" fornita da un server web in esecuzione sulla macchina dell'*attaccante* che riproduce il sito ufficiale e sulla quale è presente un pulsante "Download" che redirige ad una "risorsa di rete" sulla macchina dell'*attaccante*.   
 

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