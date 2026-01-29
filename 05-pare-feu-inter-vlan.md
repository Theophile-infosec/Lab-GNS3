# Déploiement du service DNS<br>
<br>

**Sommaire** : <br>

I. [Création du VLAN serveur & transformation du firewall en passerelle](#i-cr%C3%A9ation-du-vlan-serveur--transformation-du-firewall-en-passerelle) <br>
II. [Mise à jour DHCP pour la nouvelle architecture](https://github.com/Theophile-infosec/Lab-GNS3/blob/main/05-pare-feu-inter-vlan.md#ii-mise-%C3%A0-jour-dhcp-pour-la-nouvelle-architecture) <br>
III. [Mise à jour DNS pour la nouvelle architecture](https://github.com/Theophile-infosec/Lab-GNS3/blob/main/05-pare-feu-inter-vlan.md#iii-mise-%C3%A0-jour-dns-pour-la-nouvelle-architecture) <br>
IV. [Mise en place du DHCP relay sur le pare-feu](https://github.com/Theophile-infosec/Lab-GNS3/blob/main/05-pare-feu-inter-vlan.md#iv-mise-en-place-du-dhcp-relay-sur-le-pare-feu) <br>
V. [Définition de la politique nftables](https://github.com/Theophile-infosec/Lab-GNS3/blob/main/05-pare-feu-inter-vlan.md#v-d%C3%A9finition-de-la-politique-nftables) <br>
VI. [Vérification fonctionnelle](https://github.com/Theophile-infosec/Lab-GNS3/blob/main/05-pare-feu-inter-vlan.md#vi-v%C3%A9rification-fonctionnelle) <br>

<br>

# Objectifs :<br>
<br>

Transformer l’architecture initiale en une topologie segmentée avec :
- Un VLAN dédié au serveur (VLAN 30)
- Un pare-feu Linux jouant le rôle de passerelle inter-VLAN
- Une politique de filtrage restrictive (deny by default)
- Un routage contrôlé entre VLAN clients et VLAN serveur
- Intégration DHCP relay + DNS

**Topologie du laboratoire :**
<br>
<br>
<img width="611" height="453" alt="image" src="https://github.com/user-attachments/assets/d53416bd-ed0a-46c9-a8d2-e2302ebf31b9" />
<br>

- Serveur DHCP : Linux_Server-1 (e0)
- Clients : Linux_Desktop-1 (e0), Windows-10_Desktop-1 (e0)
- Commutation : Linux-bridge_Switch
- Pare-feu : Linux_Fierwall (e0)
- Réseau : 10.10.10.0/24<br>
<br>

# Procédures :<br>
<br>

## I. Création du VLAN serveur & transformation du firewall en passerelle

### 1. Ajout du VLAN 30 côté switch Linux

Fichier : <br>
*/usr/local/sbin/bridge-vlan.sh*

Ajout :
- VLAN 30 pour le serveur Linux
- Port trunk vers le firewall
- Ports access pour VLAN 10, 20, 30

**Objectif** :
- Segmenter physiquement et logiquement le VLAN serveur afin d’isoler les services (DHCP/DNS) des VLAN clients.<br>
<br>
Capture d'écran : <br>
<br>
<img width="513" height="213" alt="image" src="https://github.com/user-attachments/assets/09d42391-07b0-4137-9cc5-c888f2c76771" /><br>
<br>
<br>

**Vérification**

Commande :
*bridge vlan show*<br>
<br>
Capture d'écran : <br>
<br>
<img width="392" height="177" alt="image" src="https://github.com/user-attachments/assets/740a96cd-4634-4988-a36e-6984fec51d18" /><br>
<br>
<br>

### 2. Configuration des VLANs sur le firewall

Fichier : <br>
*/etc/netplan/01-netcfg.yaml*

Interfaces configurées :
- *ens3.10 → 10.10.10.254/24*
- *ens3.20 → 10.10.20.254/24*
- *ens3.30 → 10.10.30.254/24*

Le **firewall devient la gateway** par défaut de chaque VLAN.

**Objectif** :
- Centraliser le routage inter-VLAN sur le firewall afin de pouvoir appliquer un filtrage stateful.<br>
<br>
Capture d'écran : <br>
<br>
<img width="252" height="215" alt="image" src="https://github.com/user-attachments/assets/33a76094-3ed8-40b8-9451-4645b87b603c" /><br>
<br>
<br>

### 3. Activation du routage IPv4

Fichier : <br>
*/etc/sysctl.conf*

Commande : <br>
*sudo sysctl -p*

**Objectif** :
- Autoriser le transfert des paquets entre interfaces VLAN (fonctionnement L3 du firewall).<br>
<br>
Capture d'écran : <br>
<br>
<img width="182" height="29" alt="image" src="https://github.com/user-attachments/assets/a6714a7b-62a3-4bfc-97d3-d123f218baf9" /><br>
<br>
<br>

## II. Mise à jour DHCP pour la nouvelle architecture


### 1. Mise à jour de l'interface

Fichier :<br>
*/etc/default/isc-dhcp-server*

Modification :<br>
*INTERFACESv4="ens3"*

**Objectif** :
- Retirer l'écoute sur l'interface VLAN car plus de trunk<br>
<br>
Capture d'écran : <br>
<br>
<img width="161" height="40" alt="image" src="https://github.com/user-attachments/assets/15e72758-f5cd-446c-b454-9a6c0085d437" /><br>
<br>
<br>


### 2. Mise à jour des scopes

Fichier : <br>
*/etc/dhcp/dhcpd.conf*

Ajout :
- VLAN 30
- Router → 10.10.30.254
- DNS → 10.10.30.1

**Objectif** :
- Adapter la distribution IP à la nouvelle passerelle (firewall).<br>
<br>
Capture d'écran : <br>
<br>
<img width="335" height="324" alt="image" src="https://github.com/user-attachments/assets/341662cc-c7c0-411e-8aa0-e85b2011528e" /><br>
<br>
<br>

### 3. Mise à jour netplan côté serveur

Fichier : <br>
*/etc/netplan/01-static.yaml*

Modification : <br>
*via: 10.10.30.254*

**Objectif** :
- Faire du firewall la passerelle par défaut du serveur.<br>
<br>
Capture d'écran : <br>
<br>
<img width="236" height="182" alt="image" src="https://github.com/user-attachments/assets/9e596844-b4cf-4ba5-868e-4d9dfb94729b" /><br>
<br>
<br>

### 4. Suppression des anciennes interfaces VLAN

Commande : <br>
*sudo ip link delete ens3.X*

**Objectif** :
- Éviter les incohérences réseau dues aux interfaces persistantes.<br>
<br>
<br>

## III. Mise à jour DNS pour la nouvelle architecture

### 1. Vérification port 53

Commande : <br>
*ss -tulnp | grep :53*

**Objectif** :
- S’assurer que Bind écoute sur 10.10.30.1 (UDP + TCP).<br>
<br>
Capture d'écran : <br>
<br>
<img width="654" height="69" alt="image" src="https://github.com/user-attachments/assets/d9cecae0-fdab-4f04-8d99-528408026199" /><br>
<img width="665" height="66" alt="image" src="https://github.com/user-attachments/assets/d6876dcd-d3f8-4f5e-91d6-2a70dcf5365a" /><br>
<br>
<br>

### 2. Mise à jour zone directe

Fichier :<br>
*/etc/bind/db.lab.local*

Modification :<br>
- Serial incrémenté
- IP mise à jour → 10.10.30.1<br>

**Objectif** :
- Mettre à jour le serial et maintenir la résolution nom → IP.<br>
<br>
Capture d'écran : <br>
<br>
<img width="491" height="239" alt="image" src="https://github.com/user-attachments/assets/475031b5-84bd-478a-acc9-d26b7351f07b" /><br>
<br>
<br>

### 3. Mise à jour zone inverse

Commande : <br>
*sudo cp /etc/bind/db.10.10.10 /etc/bind/db.10.10.30*

Fichier :<br>
*/etc/bind/named.conf.local*

Modification :<br>
- *30.10.10.in-addr.arpa*
- */etc/bind/db.10.10.30*<br>

**Objectif** :
- Garder le bon lien vers le fichier de la zone inverse DNS.<br>
<br>
Capture d'écran : <br>
<br>
<img width="305" height="79" alt="image" src="https://github.com/user-attachments/assets/d345f28f-479a-4133-8cf3-34cb66e52234" />><br>
<br>
<br>

## IV. Mise en place du DHCP relay sur le pare-feu

### 1. Installation du service DHCP relay

Commande : <br>
*sudo apt install isc-dhcp-relay*

**Objectif** :
- Installer le service DHCP relay chargé de relayer les requêtes DHCP broadcast des clients vers le serveur situé dans un autre VLAN.<br>
<br>

### 2. Paramétrage du DHCP Relay

Commande : <br>
*sudo nano /etc/default/isc-dhcp-relay*

<br>
Capture d'écran : <br>
<br>
<img width="590" height="85" alt="image" src="https://github.com/user-attachments/assets/b6ca20fe-32da-4a4d-9e5d-d3decb564e16" /><br>
<br>

Détails :

*SERVERS*
→ Adresse IP du serveur DHCP (VLAN 30)

*INTERFACES*
→ Interfaces VLAN sur lesquelles le pare-feu écoute les requêtes DHCP :
- ens3.10 : VLAN 10 (clients Linux)
- ens3.20 : VLAN 20 (clients Windows)
- ens3.30 : VLAN 30 (communication serveur)

  
**Objectif** :
- Recevoir les requêtes DHCP des VLAN clients
- Les relayer vers le serveur DHCP
- Transmettre les réponses DHCP aux bons VLANs.<br>
<br>
<br>

### 3. Démarrage et vérification du service

Commande : <br>
*sudo systemctl restart isc-dhcp-relay
sudo systemctl status isc-dhcp-relay*<br>
<br>
Capture d'écran : <br>
<br>
<img width="757" height="85" alt="image" src="https://github.com/user-attachments/assets/dfab97ec-9d93-41be-b9c5-3d0fd56fbc14" /><br>
<br>
<br>

## V. Définition de la politique nftables

Fichier :<br>
*/etc/nftables.conf*

**Objectif** :
Mettre en place une segmentation stricte avec :
- VLAN clients isolés
- Accès contrôlé aux services
- Firewall stateful opérationnel<br>
<br>
Capture d'écran : <br>
<br>
<img width="898" height="373" alt="image" src="https://github.com/user-attachments/assets/a3ba7c98-e74f-4e7e-b346-f2eb7653ac26" /><br>
<br>
<br>

**Les règles mises en place** :

<br>

**Politique générale**
  - *policy drop* sur *input* et forward : tout trafic est bloqué par défaut sauf s’il correspond à une règle explicite.
  - *policy accept* sur *output* : le firewall peut initier du trafic sortant librement.
  - *ct state established,related accept* : autorise automatiquement les paquets appartenant à une connexion déjà autorisée (firewall stateful).

  <br>
  
**Chaîne input (trafic destiné au firewall)**
<br>
<br>
    • *iif "lo" accept*
<br>
→ Autorise le trafic local interne (loopback).
<br>
<br>
    • *iifname { "ens3.10","ens3.20" } udp dport 67 accept*
<br>
→ Autorise les requêtes DHCP des VLAN clients vers le firewall (fonction DHCP relay).
<br>
<br>
    • *iifname "ens3.30" ip saddr 10.10.30.1 udp dport 67 accept*
  <br>
→ Autorise les réponses du serveur DHCP (10.10.30.1) vers le firewall relay.
<br>
<br>
    • *iifname "ens3.30" tcp dport 22 accept*
  <br>
→ Autorise l’accès SSH d’administration depuis le VLAN30 uniquement.
<br>
<br>
**Chaîne forward (trafic inter-VLAN)**
  <br>
<br>
    • *iifname { "ens3.10","ens3.20" } oifname "ens3.30" ip daddr 10.10.30.1 udp dport 53 accept*
  <br>
→ Autorise les requêtes DNS UDP des VLAN clients vers le serveur DNS en VLAN30.
<br>
<br>
    • *iifname { "ens3.10","ens3.20" } oifname "ens3.30" ip daddr 10.10.30.1 tcp dport 53 accept*
  <br>
→ Autorise les requêtes DNS TCP (fallback ou réponses volumineuses).
<br>
<br>
    • *iifname { "ens3.10","ens3.20" } oifname "ens3.30" ip daddr 10.10.30.1 udp dport 67 accept*
  <br>
→ Autorise le trafic DHCP relay vers le serveur DHCP en VLAN30.
<br>
<br>
    • *iifname { "ens3.10","ens3.20" } oifname "ens3.30" ip daddr 10.10.30.1 ip protocol icmp accept*
  <br>
→ Autorise les tests de connectivité (ping) vers le serveur.
<br>
<br>
**Résumé des règles**<br>

- La configuration met en place une segmentation stricte entre VLAN clients (ens3.10, ens3.20) et VLAN serveurs (ens3.30).
- Seuls les flux nécessaires au fonctionnement de l’infrastructure sont autorisés : DNS, DHCP via relay et ICMP vers le serveur, ainsi que l’administration SSH depuis le VLAN serveur.
- Tout autre trafic inter-VLAN est bloqué par la politique par défaut, ce qui garantit un cloisonnement réseau minimaliste et contrôlé.<br>
<br>
<br>

## VI. Vérification fonctionnelle

### 1. Validation DHCP via relay

**Objectif** :
- Vérifier le fonctionnement du relay à travers le firewall.<br>
<br>
Capture d'écran Wireshark DHCP côté client Windows : <br>
<br>
<img width="704" height="77" alt="image" src="https://github.com/user-attachments/assets/79ad5b79-1749-4bf3-af52-e1277844eae9" /><br>
<br>
Capture d'écran Wireshark DHCP côté client Linux : <br>
<br>
<img width="696" height="74" alt="image" src="https://github.com/user-attachments/assets/a974bbb0-7ce0-4620-bbc6-a3dcfaf86e41" /><br>
<br>
<br>

### 2. Segmentation VLAN

**Objectif** :
- Vérifier l'isolation des VLAN.<br>
<br>
Capture d'écran Ping VLAN10 → VLAN20 : <br>
<br>
<img width="685" height="112" alt="image" src="https://github.com/user-attachments/assets/66ad5753-fcee-4ba1-bc8e-a5692d49f3d3" /><br>
<br>
Capture d'écran Ping VLAN20 → VLAN10 : <br>
<br>
<img width="551" height="100" alt="image" src="https://github.com/user-attachments/assets/ed3b26f0-cf97-40a0-b70b-ccd7e826e691" /><br>
<br>
<br>

### 3. Accès contrôlé au VLAN serveur

**Objectif** :
- Valider l’autorisation vers le VLAN serveur.<br>
<br>
Capture d'écran Ping VLAN10 → VLAN30 : <br>
<br>
<img width="593" height="94" alt="image" src="https://github.com/user-attachments/assets/9856298f-226a-4a44-961c-e3222fa21f17" /><br>
<br>
Capture d'écran Ping VLAN20 → VLAN30 : <br>
<br>
<img width="543" height="72" alt="image" src="https://github.com/user-attachments/assets/ea361ea1-4d7a-4788-83a8-3abbe37cca30" /><br>
<br>
<br>

### 4. Résolution DNS

**Objectif** :<br>
Valider : <br>
- DNS forward
- DNS reverse
- Cohérence IP / nom
- Prise en compte du serial<br>
<br>
Capture d'écran Forward lookup & Reverse lookup Windows : <br>
<br>
<img width="357" height="104" alt="image" src="https://github.com/user-attachments/assets/d6286118-fe0c-47cf-9ccb-f838a3ef943f" /><br>
<img width="309" height="108" alt="image" src="https://github.com/user-attachments/assets/55ee3da9-0423-4a12-aeee-7b8bafd0bc90" /><br>
<br>
Capture d'écran Forward lookup & Reverse lookup Linux : <br>
<br>
<img width="764" height="155" alt="image" src="https://github.com/user-attachments/assets/712f8674-0c3c-4d9a-9039-7ec838945f4d" /><br>
<img width="709" height="60" alt="image" src="https://github.com/user-attachments/assets/224fa8c0-3db8-4efb-9ba0-e54708b88bb3" /><br>
<br>
<br>

# Conclusion :<br>
<br>

- DHCP fonctionnel via relay
- Segmentation inter-VLAN opérationnelle
- Accès contrôlé au VLAN serveur
- DNS forward & reverse cohérent
- Firewall stateful actif
- Architecture prête pour scénarios d’attaque / défense


