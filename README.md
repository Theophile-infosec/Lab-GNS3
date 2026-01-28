# ⚠️ (EN COURS DE CRÉATION) Lab personnel GNS3 & VMware

## Objectifs

Ce laboratoire consiste à concevoir et sécuriser une infrastructure réseau virtualisée sous GNS3.

Il couvre :
- Déploiement d’un serveur DHCP et DNS.
- Segmentation du réseau via VLANs (802.1Q).
- Mise en place d’un switch Linux et d’un pare-feu inter-VLAN (nftables).
- Adaptation des services à une architecture multi-VLAN.
- *Durcissement des systèmes Linux et Windows.* (⏳​ en cours de développement)
- *Simulation d’attaques depuis Kali Linux et analyse des mécanismes de défense.* (⏳​ en cours de développement)

L’objectif est de démontrer la maîtrise des fondamentaux réseau, de la segmentation et du contrôle des flux dans une architecture structurée et sécurisée.
<br>
<br>
## Documentation technique

- [Mise en place de l’environnement du laboratoire](01-mise-en-place.md)

  1. Environnement technique
  2. Création des templates
  
- [Configuration d'un service DHCP](02-dhcp.md)
  
  1. Préparation du serveur <br>
  2. Installation du serveur DHCP <br>
  3. Configuration du serveur DHCP <br>
  4. Vérification du fonctionnement
  
- [Structuration du réseau en VLAN](03-vlan.md)
  
  1. Mise en place d’un VLAN 10 (configuration non persistante)
  2. Mise en place d’un switch Linux (Bridge)
  3. Vérification du fonctionnement de la configuration non persistante
  4. Configuration persistante du switch Linux
  5. Adaptation du serveur DHCP aux VLAN
  6. Vérification fonctionnelle
  
- [Déploiement du service DNS](04-dns.md)
  
  1. Installation d’un serveur DNS <br>
  2. Création d’une zone DNS interne

- [Ajout d'un Pare-feu inter-VLAN](05-pare-feu-inter-vlan.md)

  1. Création d'un VLAN dédié au serveur & transformation du pare-feu en passerelle unique inter-VLAN <br>
  2. Définition de la politique nftables minimale <br>
  3. Mise à jour du serveur DHCP et DNS pour le nouveau design <br>
  4. Mise en place du DHCP relay sur le pare-feu
  5. Amélioration des règles du pare-feu
  

