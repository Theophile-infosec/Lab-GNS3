# Structuration du réseau en VLAN<br>
<br>

**Sommaire** : <br>

I. [Mise en place d’un VLAN 10 (configuration non persistante)](#i-mise-en-place-dun-vlan-10-configuration-non-persistante) <br>
II. [Mise en place d’un switch Linux (Bridge)](#ii-mise-en-place-dun-switch-linux-bridge) <br>
III. [Vérification du fonctionnement de la configuration non persistante](#iii-v%C3%A9rification-du-fonctionnement-de-la-configuration-non-persistante) <br>
IV. [Configuration persistante du switch Linux](#iv-configuration-persistante-du-switch-linux) <br>
V. [Adaptation du serveur DHCP aux VLAN](#v-adaptation-du-serveur-dhcp-aux-vlan) <br>
VI. [Vérification fonctionnelle](#vi-v%C3%A9rification-fonctionnelle) <br>

<br>

# Objectifs :<br>
<br>
Mettre en place une segmentation logique du réseau à l’aide de VLAN (802.1Q) afin :

- d’isoler les flux entre groupes de machines,
- de préparer la mise en place d’un filtrage inter-VLAN,
- d’illustrer le fonctionnement du tagging 802.1Q,
- d’adapter le service DHCP à une architecture segmentée.

Deux VLANs sont définis :
- VLAN 10 → 10.10.10.0/24<br>
- VLAN 20 → 10.10.20.0/24

**Topologie du laboratoire :**
<br>
<br>
<img width="620" height="383" alt="image" src="https://github.com/user-attachments/assets/846c38dc-29f4-40fd-94c3-b5d25d6e385d" />
<br>

- Serveur DHCP : Linux_Server-1 (e0)
- Clients : Linux_Desktop-1 (e0), Windows-10_Desktop-1 (e0)
- Commutation : Linux-bridge_Switch
- Plan d'adressage : VLAN10 → 10.10.10.0/24, VLAN20 → 10.10.20.0/24<br>
<br>

# Procédures :<br>
<br>

## I. Mise en place d’un VLAN 10 (configuration non persistante)

### 1. Création d’une interface VLAN sur le serveur

Modification du fichier :<br>
*/etc/netplan/01-static.yaml*

Ajout d’une interface VLAN :
- ens3.10
- id: 10
- link: ens3
- adresse : 10.10.10.1/24

Capture d'écran : <br>
<br>
<img width="180" height="234" alt="image" src="https://github.com/user-attachments/assets/6a2a4360-74a4-445f-8aaa-93a0304687dc" /><br>
<br>
<br>

### 2. Vérification de la création de l’interface

Commande :<br>
*ip a*

**Objectifs** :
- Vérifier la présence de l’interface ens3.10
- Vérifier l’adresse IP associée<br>
<br>

Capture d'écran : <br>
<br>
<img width="662" height="180" alt="image" src="https://github.com/user-attachments/assets/dd5a5cb9-121d-4eef-be74-def620617664" /><br>
<br>
<br>

### 3. Adapter le service DHCP

Fichier :<br>
*/etc/default/isc-dhcp-server*

Modification :<br>
*INTERFACESv4="ens3.10"*

**Objectif** :
- Faire écouter le service sur l’interface VLAN.<br>
<br>

Capture d'écran : <br>
<br>
<img width="184" height="35" alt="image" src="https://github.com/user-attachments/assets/3df20d68-6302-4199-8164-9c325532f24f" /><br>
<br>
<br>

## II. Mise en place d’un switch Linux (Bridge)

Un switch logiciel est déployé afin d'assurer la commutation des trames et la segmentation du réseau.

### 1. Création d’un bridge

Commande : <br>
*sudo ip link add name br0 type bridge*

Association des interfaces : <br>
*sudo ip link set ensX master br0*

Activation :<br>
*sudo ip link set ensX up<br>
sudo ip link set br0 up*

Vérification : <br>
*bridge link*<br>
<br>

Capture d'écran : <br>
<br>

<img width="810" height="113" alt="image" src="https://github.com/user-attachments/assets/5d62aa57-4966-413d-9017-d9c3bd4ce411" /><br>
<br>
<br>

### 2. Configuration des ports VLAN

Ports configurés :
- Access VLAN 10 : ens3, ens4
- Trunk : ens8

Commandes :<br>
*sudo bridge vlan add ensX vid 10 pvid untagged<br>
sudo bridge vlan del ensX vid 1<br>
sudo bridge vlan add dev ensX vid 10*

Activation du filtrage VLAN :<br>
*ip link set br0 type bridge vlan_filtering 1*<br>
<br>

Capture d'écran : <br>
<br>
<img width="394" height="146" alt="image" src="https://github.com/user-attachments/assets/b1cd001f-fc7c-4aaa-be2e-3d99ebddbae2" /><br>
<br>
<br>

## III. Vérification du fonctionnement de la configuration non persistante

### 1. Vérification du tagging 802.1Q

Observation via Wireshark :
- Trame issue VLAN 10 → ID 10<br>
<br>

**Objectif** :
- Valider la présence du champ 802.1Q dans l’en-tête Ethernet.<br>
<br>

Capture d'écran : <br>
<br>
<img width="443" height="78" alt="image" src="https://github.com/user-attachments/assets/f413f620-dc82-4d98-a3dc-6c5a573d05c4" /><br>
<br>
<br>

### 2. Vérification côté client Linux

Commandes :
*ip a*

**Objectifs** :
- Vérifier l’attribution d’une IP dynamique dans le VLAN10<br>
<br>
Capture d'écran : <br>
<br>
<img width="805" height="200" alt="image" src="https://github.com/user-attachments/assets/f7d9d0eb-e582-4426-9515-65a09583b3ce" /><br>
<br>
<br>

### 3. Vérification de l'isolation du VLAN avec le client Windows

Commandes :
*ipconfig*

**Objectifs** :
- Confirmer que le client ne recoit aucune IP sur l'interface ens5 (présence d'une adresse APIPA)
- Confirmer l'attribution d'une IP sur l'interface ens3 <br>
<br>
Capture d'écran pour l'interface ens5 :<br>
<br>
<img width="548" height="76" alt="image" src="https://github.com/user-attachments/assets/dc8c6749-b227-4f86-b3db-71f4e88be8ed" /><br>
<br>
<br>
Capture d'écran pour l'interface ens3 :<br>
<br>
<img width="544" height="87" alt="image" src="https://github.com/user-attachments/assets/d266d5ad-8d06-49e0-bc62-940d5a4c1dcb" /><br>
<br>
<br>


## IV. Configuration persistante du switch Linux

### 1. Configuration du netplan

Fichier : <br>
*/etc/netplan/01-bridge.yaml*

**Objectifs** :
- Déclaration du bridge br0
- Association des interfaces physiques
- STP désactivé (pas de boucle possible)
- *dhcp4: false* car bridge linux = commutateur de couche 2<br>
<br>
Capture d'écran : <br>
<br>
<img width="170" height="406" alt="image" src="https://github.com/user-attachments/assets/85f9622b-e0ca-40af-88fc-bff7b638121d" /><br>
<br>
<br>

### 2. Script de configuration VLAN persistant

Fichier :  <br>
*/usr/local/sbin/bridge-vlan.sh*

**Objectifs** :
- Activer vlan_filtering
- Nettoyer VLAN 1
- Ajouter VLAN 10 et 20
- Configurer trunk<br>
<br>
Capture d'écran :<br>
<br>
<img width="368" height="383" alt="image" src="https://github.com/user-attachments/assets/f1f97836-f27c-457a-b444-0bf57e7d9a7b" /><br>
<br>
<br>

Création et démarage d’un service systemd :<br>
*sudo systemctl enable bridge-vlan.service*<br>
<br>
Capture d'écran :<br>
<br>
<img width="654" height="78" alt="image" src="https://github.com/user-attachments/assets/4e9275d7-6f25-4beb-a70b-2330633002de" /><br>
<br>
<br>


## V. Adaptation du serveur DHCP aux VLAN

### 1. Modification netplan

Modification netplan :
- ens3.10 → 10.10.10.1/24
- ens3.20 → 10.10.20.1/24<br>
<br>
Capture d'écran :<br>
<br>
<img width="160" height="210" alt="image" src="https://github.com/user-attachments/assets/9a6f331a-0915-4b0e-b7c3-2f9724917519" /><br>
<br>
<br>

### 2. Modification des interfaces

Fichier :<br>
*/etc/default/isc-dhcp-server*

Modification :<br>
*INTERFACESv4="ens3.10 ens3.20"*<br>
<br>
Capture d'écran :<br>
<br>
<img width="248" height="35" alt="image" src="https://github.com/user-attachments/assets/5f7030c4-39bc-499e-8ae8-c5141af227a7" /><br>
<br>
<br>

### 3. Ajout du subnet VLAN 20

Fichier : <br>
*/etc/dhcp/dhcpd.conf*

Ajout : <br>
*subnet 10.10.20.0 netmask 255.255.255.0 {<br>
  range 10.10.20.100 10.10.20.200;<br>
  option routers 10.10.20.1;<br>
  option broadcast-address 10.10.20.255<br>
}*<br>
<br>
Capture d'écran :<br>
<br>
<img width="331" height="213" alt="image" src="https://github.com/user-attachments/assets/f7a36f21-3328-48b8-a3c6-06d84fc66018" /><br>
<br>
<br>

## VI. Vérification fonctionnelle

### 1. Redémarrage du service

Commande : <br>
*sudo systemctl restart isc-dhcp-server*<br>
*sudo systemctl status isc-dhcp-server*

<br>
Capture d'écran :<br>
<br>
<img width="482" height="97" alt="image" src="https://github.com/user-attachments/assets/c60b6fa3-e2db-4376-876b-dc723570b0a8" /><br>
<br>
<br>

### 2. Vérification de l'attribution DHCP

Commandes :  <br>
*ip a<br>
ip config*

Linux (VLAN 10) :
- IP obtenue : 10.10.10.101

Windows (VLAN 20) :
- IP obtenue : 10.10.20.100<br>
<br>
Captures d'écran :<br>
<br>
<img width="379" height="22" alt="image" src="https://github.com/user-attachments/assets/1f709912-e532-485f-822a-8ea0ad2ba384" /><br>
<img width="441" height="15" alt="image" src="https://github.com/user-attachments/assets/81b6e8d7-8933-45f2-a8d2-4c91386e4620" /><br>
<br>
<br>

### 3. Vérification de l'isolation inter-VLAN

Commandes : <br>
*ping 10.10.20.100<br>
ping 10.10.10.101*

**Objectifs** :
- Confirmer qu'il n'y a aucun échange inter-VLAN
- Confirmer que la ségmentation est fonctionnelle<br>
<br>
Captures d'écran :<br>
<br>
<img width="705" height="117" alt="image" src="https://github.com/user-attachments/assets/3a33f7f6-552c-454a-a66e-c1737824c907" /><br>
<img width="537" height="131" alt="image" src="https://github.com/user-attachments/assets/e46e94dc-5d00-4c4b-bf68-24c70f547c18" /><br>
<br>
<br>

### 4. Vérification du tagging 802.1Q

Observation via Wireshark : 
- Trame issue VLAN 10 → ID 10
- Trame issue VLAN 20 → ID 20

Objectif :
- Valider la présence du champ 802.1Q dans l’en-tête Ethernet.<br>
<br>
Captures d'écran :<br>
<br>
<img width="371" height="23" alt="image" src="https://github.com/user-attachments/assets/54037dc9-fc45-442d-be3c-90adf8cfa316" /><br>
<img width="374" height="22" alt="image" src="https://github.com/user-attachments/assets/82b5d300-510c-4317-a87e-eae6ef43944a" /><br>
<br>
<br>


# Conclusion :<br>
<br>

La segmentation réseau est opérationnelle :
- VLAN 10 et VLAN 20 sont correctement isolés.
- Le tagging 802.1Q est observable dans les trames Ethernet.
- Le serveur DHCP distribue dynamiquement des adresses distinctes par VLAN.
- L’isolation inter-VLAN est confirmée par absence de communication transversale.

Cette architecture est désormais structurée pour l’intégration d’un service DNS, configuré dans le chapitre suivant.

