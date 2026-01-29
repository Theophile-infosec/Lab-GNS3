# Déploiement du service DNS<br>
<br>

# Objectifs :<br>
<br>

Déployer un serveur DNS interne (Bind9) afin de :
- Fournir une résolution de noms interne au laboratoire.
- Centraliser la résolution DNS pour les VLAN 10 et VLAN 20.
- Distribuer automatiquement le serveur DNS via DHCP.
- Mettre en place une zone directe et une zone inverse.
- Valider la cohérence des enregistrements A et PTR.

Le serveur DNS est hébergé sur le serveur Linux principal.

**Topologie du laboratoire :**
<br>
<br>
<img width="620" height="383" alt="image" src="https://github.com/user-attachments/assets/846c38dc-29f4-40fd-94c3-b5d25d6e385d" />
<br>

- Serveur DHCP : Linux_Server-1 (e0)
- Clients : Linux_Desktop-1 (e0), Windows-10_Desktop-1 (e0)
- Commutation : Linux-bridge_Switch
- Réseau : 10.10.10.0/24<br>
<br>

# Procédures :<br>
<br>

## I. Installation et configuration initiale de Bind9

### 1. Installation du service

Paquets installés :<br>
*sudo apt update && sudo apt install -y bind9 bind9-utils dnsutils*

**Objectifs** :
- Installer le daemon named
- Installer les outils de diagnostic (dig, named-checkconf, named-checkzone)

### 2. Configuration générale

Fichier :<br>
*/etc/bind/named.conf.options*

Paramètres principaux :
- *recursion yes;* → autorise les requêtes récursives.
- *allow-query { any; };* → autorise les clients internes.
- *dnssec-validation auto;* → validation DNSSEC.
- *listen-on { any; };* → écoute sur toutes les interfaces IPv4.
- *listen-on-v6 { any; };* → écoute IPv6.

Raison :
- Dans un environnement de laboratoire interne, l’ouverture sur toutes les interfaces simplifie la configuration multi-VLAN.<br>
<br>
Capture d'écran : <br>
<br>
<img width="204" height="124" alt="image" src="https://github.com/user-attachments/assets/277c9d05-2bb4-476d-aacf-14b2055a5dce" /><br>
<br>
<br>

### 3. Vérification de la configuration

Commandes :
- *sudo named-checkconf*  → contrôle syntaxique
- *sudo systemctl restart bind9*  → redémarrage
- *sudo systemctl status bind9*  → vérification<br>
<br>
Capture d'écran : <br>
<br>
<img width="686" height="99" alt="image" src="https://github.com/user-attachments/assets/792d41df-6cb5-4000-8844-a1dcce67b184" /><br>
<br>
<br>

### 4. Vérification des ports d’écoute

Bind9 doit écouter sur le port 53 :
- UDP → requêtes classiques
- TCP → transferts de zone / réponses volumineuses

Commande : <br>
*ss -tulnp | grep :53*

Explication :
- UDP → état UNCONN (socket prêt à recevoir)
- TCP → état LISTEN (attente de connexions)<br>
<br>
Capture d'écran pour le protocole UDP : <br>
<br>
<img width="642" height="82" alt="image" src="https://github.com/user-attachments/assets/b55c93ae-de3d-441f-8ddb-0f06b40b8c1a" /><br>
<br>
<br>
Capture d'écran pour le protocole TCP : <br>
<br>
<img width="496" height="128" alt="image" src="https://github.com/user-attachments/assets/d73caaae-16a4-4415-b2e9-90ab084f643c" /><br>
<br>
<br>

### 5. Test de fonctionnement local

Commande : <br>
*dig @127.0.0.1 localhost*

**Objectifs** :
- Vérifier que le service répond.
- Confirmer le statut NOERROR.<br>
<br>
Capture d'écran : <br>
<br>
<img width="586" height="138" alt="image" src="https://github.com/user-attachments/assets/76a14fc7-a84a-46b1-a535-c063202cea05" /><br>
<br>
<br>

## II. Distribution du DNS via DHCP

Fichier : <br>
*/etc/dhcp/dhcpd.conf*

Ajouts : <br>
*option domain-name-servers 10.10.10.1;<br>
option domain-name-servers 10.10.20.1;*

**Objectif** :
- Distribuer le serveur DNS correspondant à chaque VLAN.

Vérification côté client :
- Linux → *resolvectl status*
- Windows → *ipconfig /all*<br>
<br>
Capture d'écran : <br>
<br>
<img width="353" height="49" alt="image" src="https://github.com/user-attachments/assets/55e7677e-bd25-4764-a85a-23d600ad570f" /><br>
<img width="433" height="26" alt="image" src="https://github.com/user-attachments/assets/2f76cb7a-a674-41be-b20b-148cdcbba613" /><br>
<br>
<br>

## III. Création d’une zone DNS interne

### 1. Déclaration de la zone directe

Zone définie :<br>
*lab.local*

Fichier :<br>
*/etc/bind/named.conf.local*

Ajouts :<br>
*zone "lab.local" {<br>
    type master;<br>
    file "/etc/bind/db.lab.local";<br>
};*

Explication :<br>
- */etc/bind/db.lab.local* → fichier de la zone directe DNS, contient toutes les règles de correspondance<br>
<br>
Capture d'écran : <br>
<br>
<img width="312" height="81" alt="image" src="https://github.com/user-attachments/assets/668ef42c-11eb-4705-8086-8c8ea519dc0e" /><br>
<br>
<br>

### 2. Fichier de zone directe

Fichier :<br>
*/etc/bind/db.lab.local*

Explications :<br>
- *ns.lab.local* → serveur faisant autorité.
- *admin.lab.local* → contact admin (SOA : Start Of Authority).
- *Enregistrements A* → résolution nom → IP.<br>
<br>
Capture d'écran : <br>
<br>
<img width="494" height="242" alt="image" src="https://github.com/user-attachments/assets/410765be-b86a-4b57-a57c-5c512e657887" /><br>
<br>
<br>

### 3. Vérification de la zone

Commande : <br>
*sudo named-checkzone lab.local /etc/bind/db.lab.local*<br>
<br>
Capture d'écran : <br>
<br>
<img width="595" height="49" alt="image" src="https://github.com/user-attachments/assets/784c9243-9556-4e0e-9495-7a98fe37ccd5" /><br>
<br>
<br>

### 4. Test de résolution directe

Commande : <br>
*dig server.lab.local*

Objectif :
- Vérifier la résolution A → IP.<br>
<br>
Capture d'écran : <br>
<br>
<img width="570" height="185" alt="image" src="https://github.com/user-attachments/assets/1238b067-26e9-448c-97fd-0a8167abe1d8" /><br>
<br>
<br>

## IV. Création de la zone inverse

### 1. Déclaration de la zone inverse

Fichier : <br>
*/etc/bind/named.conf.local*

Ajout : <br>
*zone "10.10.10.in-addr.arpa" {<br>
    type master;<br>
    file "/etc/bind/db.10.10.10";<br>
};*<br>

Explication :<br>
- */etc/bind/db.10.10.10* → fichier de la zone inverse DNS pour le réseau 10.10.10.0/24<br>
<br>
Capture d'écran : <br>
<br>
<img width="304" height="75" alt="image" src="https://github.com/user-attachments/assets/f262bc56-5cb5-430a-9c19-06b99a921b8e" /><br>
<br>
<br>

### 2. Fichier de zone inverse

Fichier : <br>
*/etc/bind/db.10.10.10*

Explication :<br>
- *1   IN PTR server.lab.local.* → L’adresse 10.10.10.1 correspond au nom server.lab.local.<br>
<br>
Capture d'écran : <br>
<br>
<img width="493" height="183" alt="image" src="https://github.com/user-attachments/assets/81421336-f426-4896-a7ba-ec67b531fbc6" /><br>
<br>
<br>

### 3. Vérification inverse

Commande : <br>
*dig -x 10.10.10.1*<br>
<br>
Capture d'écran : <br>
<br>
<img width="584" height="209" alt="image" src="https://github.com/user-attachments/assets/02d6fa7b-1c24-49bc-84c6-d5fe33b764b9" /><br>
<br>
<br>

## V. Validation côté clients

Commandes :<br>
*dig -x 10.10.10.1* → Linux<br>
*nslookup 10.10.10.1* → Windows<br>
<br>
Capture d'écran du client Linux : <br>
<br>
<img width="688" height="332" alt="image" src="https://github.com/user-attachments/assets/5ed6221b-1dd9-4cda-997e-6d5d510bad4c" /><br>
<br>
Capture d'écran du client Windows : <br>
<br>
<img width="307" height="101" alt="image" src="https://github.com/user-attachments/assets/7ea19cee-d018-4fca-979f-0a75a4bbd9e4" /><br>
<br>
<br>

# Conclusion :<br>
<br>

Le service DNS interne est opérationnel :
- Bind9 est installé et actif.
- Le port 53 est ouvert en UDP et TCP.
- Une zone directe (lab.local) est configurée.
- Une zone inverse (10.10.10.in-addr.arpa) est fonctionnelle.
- Les clients des VLAN 10 et 20 utilisent automatiquement ce DNS via DHCP.
- Les résolutions directe (A) et inverse (PTR) sont validées.

Le réseau dispose désormais :
- d’une attribution IP dynamique (DHCP),
- d’une résolution interne cohérente (DNS),
- d’une segmentation réseau fonctionnelle (VLAN),<br>

préparant l’intégration d’un filtrage inter-VLAN contrôlé dans le chapitre suivant.
