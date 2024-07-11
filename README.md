![foto](https://github.com/user-attachments/assets/7b7d3808-5db1-4b94-bc23-447f5c2e67d2)

# Introduzione
Formalmente, l'ARP Poisoning (o ARP spoofing) è un attacco di rete in cui un malintenzionato invia falsi messaggi ARP (Address Resolution Protocol) per associare il proprio indirizzo MAC all'indirizzo IP di un altro dispositivo, permettendogli di intercettare, modificare o interrompere il traffico di rete tra i dispositivi target.

# Premesse
Per effettuare un attacco di questo tipo, bisogna creare un ambiente ISOLATO in cui configurare due VMs Kali Linux (KaliAttacker e KaliVictim), uno Switch per connettere le due macchine e un Router, il quale sarà il default gateway per entrambi i dispositivi. 

# Configurazione VM e topologia
Innanzitutto, bisogna configurare l'ambiente.
Per quanto riguarda le macchine virtuali, utilizziamo una piattaforma di virtualizzazione (personalmente ho utilizzato VirtualBox, ma qualsiasi sw è ben accetto) per creare le macchine virtuali necessarie all'attacco.

Aggiungiamo le macchine virtuali dalla sezione 'Preferences' in 'Edit', andiamo su 'VirtualBox VMs' (1) e clicchiamo su 'New' (2), 'Next' e poi selezioniamo la nostra macchina virtuale (3), dopodiché clicchiamo Finish (4):
![1](https://github.com/user-attachments/assets/63fa5847-2699-49fb-9f54-ea50d4cb868c)


Ora andiamo alla sezione Edit premendo l'apposito pulsante in basso a destra (5). 
Si aprirà un menù a tendina, e dobbiamo spuntare 'Allow GNS3 to use any configured VirtualBox adapter' nella sezione Network (6). In questo modo, GNS3 ha il permesso di utilizzare qualsiasi adattatore di rete configurato nella VM in VirtualBox Questo include adattatori in modalità bridge, NAT, host-only, ecc. Abilitando questa opzione, GNS3 può automaticamente scegliere l'adattatore di rete più appropriato disponibile per la topologia, offrendo maggiore flessibilità nella configurazione delle connessioni di rete tra i nodi virtuali. 
Premiamo 'OK' per confermare le modifiche (7):
![2](https://github.com/user-attachments/assets/54cfdf4c-d1a1-424e-b62b-253090af7fe2)

Fatto ciò, usciamo dal menù Edit premendo 'Apply' e successivamente 'OK'.



Adesso siamo pronti ad importare le nostre macchine virtuali nella topologia di rete.
Ipotizzando di avere già creato un progetto, aggiungiamo le VMs alla nostra topologia con un semplice Drag&Drop sul pannello del progetto.
Questo sarà il risultato:
![3](https://github.com/user-attachments/assets/ca8b4f73-aeeb-4e99-851a-13946d721b4e)


Aggiungiamo allo stesso modo uno Switch e un Router e colleghiamo tra loro le componenti.
NOTA: Per questo esempio, le configurazioni sono le seguenti:
-  KaliVictim:  eth0 < = > eth0 Switch
-  KaliAttacker:  eth0 < = > eth1 Switch
-  Switch:  eth0 < = > eth0 KaliVictim
            eth1 < = > eth0 KaliAttacker
            eth2 < = > fa0/0 Router
-  Router:  fa0/0 < = > eth2 Switch

A questo punto non resta che avviare tutti i nodi premendo play in alto:
![4](https://github.com/user-attachments/assets/54d6d9ea-362d-4b38-8851-7a390f663fdc)

Si avvieranno automaticamente anche le VMs, e ora non resta che impostare i rispettivi indirizzi statici.



Per fare ciò, andiamo nel terminale delle singole VM e digitiamo il seguente comando:
```bash
sudo /etc/network/interfaces
```

In questo modo si accede al file che definisce tutte le interfacce di rete della macchina.
Aggiungiamo dunque un nuovo record all'interno di questo file :
```bash
auto eth0
iface eth0 inet static
            address 192.168.1.101 #IP Vittima
            netmask 255.255.255.0
            gateway 192.168.1.1   #IP fa0/0 Router
```
Analogo per l'attaccante, ma con IP 192.168.1.102 (sono IP di esempio).

Salviamo e chiudiamo il file. Riavviamo l'interfaccia di rete con il seguente comando:
```bash
sudo systemctl restart networking
```

Per verificare la modifica, andiamo a digitare il seguente comando:
```bash
ifconfig
```

Se è andato tutto bene, dovremmo vedere che eth0 è configurato con l'IP che le abbiamo dato.
![5](https://github.com/user-attachments/assets/3e7388b9-be7f-4a43-9683-dfdebd1179a5)
![6](https://github.com/user-attachments/assets/e9594b21-5c24-402d-8f63-80167814276e)

NOTA: Facciamo molta attenzione al MAC address dell'attaccante, perché servirà notarlo durante l'esecuzione dell'attacco:
```bash
ether 08:00:27:a3:d5:48
```


# Configurazione Router
Per quanto riguarda il Router, dobbiamo configurare la sua interfaccia fa0/0 in modo che abbia l'indirizzo IP che abbiamo impostato.
Per fare ciò dobbiamo aprire la console del Router da GNS3 (facendo doppio click sull'icona oppure tasto destro, 'Console') ed eseguire una serie di comandi, che ho riportato nel seguente screenshot:
![7](https://github.com/user-attachments/assets/18afbe0f-b89d-4772-9ee8-8f1299977caf)

Come possiamo notare, l'IP di fa0/0 è stato impostato correttamente, e ora le macchine virtuali riescono a vedere il router.
Per confermare ciò, basta fare un ping al router e osservare che effettivamente è raggiungibile:
![8](https://github.com/user-attachments/assets/718207c0-d33d-4dc0-be61-8e132c3714e5)
![9](https://github.com/user-attachments/assets/d4db4546-3ff4-48a3-a727-273438d8eb60)


# Esecuzione dell'attacco

Riprendiamo la topologia di rete:

![10](https://github.com/user-attachments/assets/16ed1759-a90d-4968-9dd5-6ea71bc6a011)

Ora apriamo la macchina virtuale dell'attaccante (KaliAttacker), apriamo un terminale e digitiamo:
```bash
ettercap -G  #Per aprire ettercap in modalità grafica
```
Vedremo aprirsi una finestra sul Desktop. Clicchiamo sulla spunta per far partire lo sniffing:
![11](https://github.com/user-attachments/assets/d03b590d-eba5-4f68-a926-bbaab59c09c3)

Ora apriamo apriamo il menù a tendina cliccando sui tre pallini e selezioniamo e 'Hosts', poi 'Scan for hosts'.
Dopo la scansione, riapriamo il menù a tendina e clicchiamo su 'Hosts', dunque 'Hosts list'.
Ecco che troveremo due host, uno è il router (Router, 192.168.1.1), l'altro è la vittima (KaliVictim, 192.168.1.101):
![12](https://github.com/user-attachments/assets/71316649-899e-4dc7-b76f-936a87a89bfa)

Selezionamo la macchina virtuale KaliVictim e impostiamolo come Target 1. Aggiungiamo invece il Router come Target 2.
Ora clicchiamo sull'icona del globo in alto a destra, selezioniamo 'ARP Poisoning' e poi premiamo 'OK':
![13](https://github.com/user-attachments/assets/ed57bb0c-6e42-45b1-9f9c-aab5922afd9e)

A questo punto l'attacco ha inizio!

![14](https://github.com/user-attachments/assets/f70f4c20-7de5-4f8d-984b-0cc5d7e55c76)



Vediamo ora nel dettaglio cosa succede durante l'attacco...


# Analisi dell'attacco

Per poter osservare gli effetti dell'attacco, abbiamo bisogno di un software che ci consenta di fare Sniffing sulle singole interfacce di rete (nel nostro caso eth0 per entrambe le VM) e di analizzare i pacchetti. Prendiamo come esempio Wireshark.
Avviamo Wireshark sia sulla macchina attaccante sia sulla macchina vittima, selezioniamo l'interfaccia eth0 e avviamo la cattura.


**KaliAttacker:**

![15](https://github.com/user-attachments/assets/014efe01-c880-4f62-9329-2c3f5bbfc1e2)
Vediamo chiaramente una coppia di pacchetti che viene periodicamente inviata dall'attaccante.

Il primo pacchetto serve per far sì che la vittima sia convinta che il router possegga come MAC Address 08:00:27:a3:d5:48, che è in realtà il MAC Address della scheda di rete di KaliAttacker.
In questo modo, tutto il traffico che dovrebbe essere indirizzato al router passa prima per l'attaccante, e poi arriva tramite lui al router.
![16](https://github.com/user-attachments/assets/d6fc360d-b0a4-482e-b5e1-03a4eedd27b3)

Il secondo pacchetto, invece, server per far sì che il router sia convinto che la vittima possegga come MAC Address 08:00:27:a3:d5:48, che è in realtà il MAC Address della scheda di rete di KaliAttacker.
In questo modo, tutto il traffico che dovrebbe essere inviato direttamente a KaliVictim passa prima per KaliAttacker, e poi arriva tramite lui al router.
![17](https://github.com/user-attachments/assets/e08223cf-786a-4d95-b0a9-52467edc143d)



**KaliVictim:**

![18](https://github.com/user-attachments/assets/12319053-c881-4a69-bc1b-c625bc11935a)
La vittima, invece, riceve periodicamente lo stesso pacchetto, che è inviato dall'attaccante per indurla a credere che il router abbia 08:00:27:a3:d5:48 come MAC Address, che sappiamo essere il MAC Address della scheda di rete di KaliAttacker. 
Le conseguenze le abbiamo espresse poc'anzi.


# Verifica dell'attacco

Per verificare la riuscita dell'attacco, proviamo a effettuare un ping da KaliVictim verso il router:
```bash
ping 192.168.1.1
```

**KaliVictim:**
La vittima è convinta di pingare il router, ma come possiamo vedere sta parlando direttamente con l'attaccante:
![19](https://github.com/user-attachments/assets/b4588c1c-ba12-41e2-879e-335275933c18)
![20](https://github.com/user-attachments/assets/b85d374e-e1d8-49b1-9e67-b7783c59e3a3)



**KaliAttacker:**
L'attaccante, d'altro canto, risponde al ping come se fosse il router. 
Notare come l'attaccante faccia un ping a sua volta verso il router, per poter poi rispondere alla vittima:
![21](https://github.com/user-attachments/assets/8e6c0577-2c3e-4098-b4b4-6708bc259220)
![22](https://github.com/user-attachments/assets/911680c0-5b52-47e9-b06b-98e23546c955)
![23](https://github.com/user-attachments/assets/0b19a438-0f5e-4c33-a357-f140d7759430)




Un modo più formale per notare che l'attacco ha avuto effetto è andare a valutare le tabelle ARP di KaliAttacker e KaliVictim.

**KaliVictim:**

![24](https://github.com/user-attachments/assets/2480df7f-5f0c-4a15-9d42-085a029ef7ae)

La vittima si ritrova con due indirizzi IP diversi associati allo stesso MAC Address.



**KaliAttacker:**

![25](https://github.com/user-attachments/assets/9054ab71-f069-46c6-b8f5-570cdc8ee85e)

D'altro canto, la tabella ARP dell'attaccante non viene in alcun modo modificata durante l'attacco.


