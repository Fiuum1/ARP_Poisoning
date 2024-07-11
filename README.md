# ARP_Poisoning


# Introduzione
Formalmente, l'ARP Poisoning (o ARP spoofing) è un attacco di rete in cui un malintenzionato invia falsi messaggi ARP (Address Resolution Protocol) per associare il proprio indirizzo MAC all'indirizzo IP di un altro dispositivo, permettendogli di intercettare, modificare o interrompere il traffico di rete tra i dispositivi target.

# Premesse
Per effettuare un attacco di questo tipo, bisogna creare un ambiente ISOLATO in cui configurare due VMs Kali Linux (KaliAttacker e KaliVictim), uno Switch per connettere le due macchine e un Router, il quale sarà il default gateway per entrambi i dispositivi. 

# Set-up generale
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
