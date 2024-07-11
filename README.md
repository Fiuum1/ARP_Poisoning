# ARP_Poisoning


# Introduzione
Formalmente, l'ARP Poisoning (o ARP spoofing) è un attacco di rete in cui un malintenzionato invia falsi messaggi ARP (Address Resolution Protocol) per associare il proprio indirizzo MAC all'indirizzo IP di un altro dispositivo, permettendogli di intercettare, modificare o interrompere il traffico di rete tra i dispositivi target.

# Premesse
Per effettuare un attacco di questo tipo, bisogna creare un ambiente ISOLATO in cui configurare due VMs Kali Linux (KaliAttacker e KaliVictim), uno Switch per connettere le due macchine e un Router, il quale sarà il default gateway per entrambi i dispositivi. 

# Set-up dell'ambiente
Innanzitutto, bisogna configurare l'ambiente.
Per quanto riguarda le macchine virtuali, utilizziamo una piattaforma di virtualizzazione (personalmente ho utilizzato VirtualBox, ma qualsiasi sw è ben accetto) per creare le macchine virtuali necessarie all'attacco.

Aggiungiamo le macchine virtuali dalla sezione 'Preferences' in 'Edit', andiamo su 'VirtualBox VMs' e clicchiamo su 'New', 'Next' e poi selezioniamo la nostra macchina virtuale (una per volta):
![1](https://github.com/user-attachments/assets/b43d331a-0d60-4d6f-8fc7-6a597801200e)

Dopodiché clicchiamo Finish, e andiamo alla sezione Edit premendo l'apposito pulsante in basso a destra:
![2](https://github.com/user-attachments/assets/1de4165c-0dff-4e17-98d9-deb9e0410713)


Si aprirà un menù a tendina, e dobbiamo spuntare "Allow GNS3 to use any configured VirtualBox adapter". In questo modo, GNS3 ha il permesso di utilizzare qualsiasi adattatore di rete configurato nella VM in VirtualBox Questo include adattatori in modalità bridge, NAT, host-only, ecc. Abilitando questa opzione, GNS3 può automaticamente scegliere l'adattatore di rete più appropriato disponibile per la topologia, offrendo maggiore flessibilità nella configurazione delle connessioni di rete tra i nodi virtuali:
![3](https://github.com/user-attachments/assets/9a7dcc42-6ee7-474a-a17a-40c0fe506b7c)

