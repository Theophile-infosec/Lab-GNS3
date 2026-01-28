# II. Installation et configuration du service DHCP
<br>
<br>
## A. Objectifs :

Mettre en place un serveur DHCP sous Ubuntu Server afin d’automatiser l’attribution des adresses IPv4 aux clients du réseau interne.

Le service devra :

- Distribuer des adresses IP dynamiques dans le sous-réseau 10.10.10.0/24 <br>
- Fournir une passerelle par défaut <br>
- Définir un serveur DNS <br>
- Attribuer un temps de bail cohérent <br>
- Permettre la vérification du processus DORA

**Topologie du laboratoire :**
<br>
<br>
<img width="576" height="352" alt="image" src="https://github.com/user-attachments/assets/74068d36-6ebd-4589-b5cb-339e9c60458c" /><br>
<br>

- Serveur DHCP : Ubuntu_Server-1 (e0)
- Clients : Ubuntu_Desktop-1 (e0), Windows-10_Desktop-1 (e0)
- Commutation : Switch1
- Réseau : 10.10.10.0/24
<br>
<br>
## B. Préparation du serveur :

### 1. Attribution d’une adresse IP statique

Avant l’installation du service DHCP, le serveur doit disposer d’une adresse IP fixe.

Raisons : 
- Les clients doivent pouvoir identifier de manière constante l’équipement qui distribue les adresses.
- L’adresse du serveur est utilisée comme passerelle par défaut.
- Risque d’incohérence réseau en cas de changement d’IP.

Le service DHCP ne peut pas s’auto-attribuer une adresse dynamique fiable avant son propre démarrage.

Fichier modifié : */etc/netplan/01-static.yaml*

Configuration appliquée :

- Interface : ens3 <br>
- Adresse IP : 10.10.10.1/24 <br>
- Passerelle : 10.10.10.254
- DNS : 8.8.8.8

Commande d’application : *sudo netplan apply*

Capture d'écran : <br>
<br>
<img width="230" height="206" alt="image" src="https://github.com/user-attachments/assets/68b70ac5-6b58-4194-a7a0-27bce348d51b" /> <br>
<br>
<br>


### 2. Sécurisation des permissions du fichier

Objectif : limiter l’exposition du fichier de configuration.

Commande : *sudo chmod 600 /etc/netplan/01-static.yaml*

Vérification : *ls -l /etc/netplan/01-static.yaml*

Capture d'écran : <br>
<br>
<img width="563" height="106" alt="image" src="https://github.com/user-attachments/assets/e842c99c-0aed-4756-a9b5-5e8e8934a70b" /><br>
<br>
<br>

### 3. Accès Internet temporaire

Un accès NAT temporaire est configuré afin de permettre l’installation des paquets nécessaires.

Objectif :

- Télécharger les dépendances <br>
- Installer le serveur DHCP

Capture d'écran : <br>
<br>
<img width="451" height="104" alt="image" src="https://github.com/user-attachments/assets/1184c5dd-7ccd-40a8-80cd-0a58562efe19" /><br>
<br>
<br>



## C. Installation du serveur DHCP :

### 1. Mise à jour des paquets

Commande : *sudo apt update*


### 2. Installation du service

Commande : *sudo apt install isc-dhcp-server*

Objectif :

- Installer le daemon DHCP <br>
- Préparer le service systemd
<br>
<br>


## D. Configuration du serveur DHCP :

### 1. Déclaration de l’interface d’écoute

Fichier : */etc/default/isc-dhcp-server*

Configuration :

INTERFACESv4="ens3" <br>
INTERFACESv6=""

Objectif :

- Spécifier sur quelle interface le service doit écouter.

Capture d'écran : <br>
<br>
<img width="174" height="35" alt="image" src="https://github.com/user-attachments/assets/a7ad5bcd-a019-492c-a19d-5d10704d3555" /><br>
<br>
<br>


### 2. Configuration du pool DHCP

Fichier : */etc/dhcp/dhcpd.conf*

Configuration appliquée :

*subnet 10.10.10.0 netmask 255.255.255.0 {<br>
  range 10.10.10.100 10.10.10.200;<br>
  option routers 10.10.10.1;<br>
  option domain-name-servers 8.8.8.8;<br>
  default-lease-time 600;<br>
  max-lease-time 7200;<br>
}*

Paramètres définis :

- Plage dynamique : 100–200 <br>
- Passerelle : 10.10.10.1 <br>
- DNS : 8.8.8.8 <br>
- Temps de bail court pour environnement de test

Capture d'écran : <br>
<br>
<img width="349" height="115" alt="image" src="https://github.com/user-attachments/assets/a22c9cb0-776e-40fc-a0ab-2f7be6f5f0e2" /><br>
<br>
<br>

## E. Activation du service :

Redémarrage : *sudo systemctl restart isc-dhcp-server*

Vérification : *sudo systemctl status isc-dhcp-server*

Objectif :

- Appliquer la configuration <br>
- Vérifier que le service est actif (running)

Capture d'écran : <br>
<br>
<img width="786" height="79" alt="image" src="https://github.com/user-attachments/assets/f5152346-5254-4adc-ad5c-1cc935c5d4e5" /><br>
<br>
<br>

## F. Vérification du fonctionnement :

### 1. Vérification côté client Linux

Commandes :

*ip a<br>
ip route<br>
ping 10.10.10.1*

Objectifs :

- Vérifier l’attribution d’une IP dynamique<br>
- Vérifier la route par défaut<br>
- Vérifier la connectivité avec le serveur

Capture d'écran : <br>
<br>
<img width="1768" height="597" alt="image" src="https://github.com/user-attachments/assets/d55166ae-ab62-4a63-9db2-1746fe4dabf2" /><br>
<br>
<br>

### 2. Vérification côté serveur (Linux)

Commande : *journalctl -u isc-dhcp-server*

Objectif :

Observer le processus DORA :

- DHCPDISCOVER<br>
- DHCPOFFER<br>
- DHCPREQUEST<br>
- DHCPACK

Capture d'écran : <br>
<br>
<img width="523" height="63" alt="image" src="https://github.com/user-attachments/assets/a2b767c7-4599-4d75-9630-46baaf93a285" /><br>
<br>
<br>

### 3. Vérification côté client Windows

Commandes : 

*ipconfig<br>
ping 10.10.10.1<br>
ping 10.10.10.101*

Objectifs :

- Vérifier l’attribution d’une IP dynamique<br>
- Vérifier la passerelle<br>
- Vérifier la connectivité inter-clients

Capture d'écran : <br>
<br>
<img width="917" height="489" alt="image" src="https://github.com/user-attachments/assets/007398c4-b89f-4573-8c95-6d270c9cb14f" /><br>
<br>
<br>


### 4. Vérification côté serveur (Windows DORA)

Commande : *journalctl -u isc-dhcp-server*

Objectif :

- Confirmer le cycle DORA pour le client Windows.
- 
Capture d'écran : <br>
<br>
<img width="519" height="65" alt="image" src="https://github.com/user-attachments/assets/4e19e1a3-5b1b-4681-a549-24daf8f6df09" /><br>
<br>
<br>

## G. Conclusion :

À l’issue de cette étape :

- Le serveur DHCP est installé et opérationnel.<br>
- La plage d’adresses 10.10.10.100–10.10.10.200 est correctement distribuée.<br>
- Les clients Linux et Windows obtiennent une adresse IP dynamique valide.<br>
- La passerelle par défaut et le DNS sont correctement transmis.<br>
- Le cycle DORA est observable dans les logs du serveur.

Le service DHCP est désormais fonctionnel et prêt à être intégré dans une architecture segmentée (VLAN) dans le chapitre suivant.
