# ⚠️ (EN COURS DE CRÉATION) Lab personnel GNS3 & VMware

## Objectifs

## Outils

## Documentation technique

- [Installation](01-installation.md)

  1. Mise en place & installation des machines virtuelles
  
- [Configurer un service DHCP](02-dhcp.md)
  
  1. Installation et configuration d'un service DHCP <br>
  2. Vérification du fonctionnement du service DHCP
  
- [Structuration du réseau en VLAN](03-vlan.md)
  
  1. Teste d’une configuration non persistante pour le VLAN 10 <br>
  2. Déploiement d’une configuration persistante & ajout du VLAN 20 <br>
  3. Adaptation de la configuration VLAN et du service DHCP côté serveur
  
- [Déploiement du service DNS](04-dns.md)
  
  1. Installation d’un serveur DNS <br>
  2. Création d’une zone DNS interne

- [Ajout d'un Pare-feu inter-VLAN](05-pare-feu-inter-vlan.md)

  1. Création d'un VLAN dédié au serveur & transformation du pare-feu en passerelle unique inter-VLAN <br>
  2. Définition de la politique nftables minimale <br>
  3. Mise à jour du serveur DHCP et DNS pour le nouveau design <br>
  4. Mise en place du DHCP relay sur le pare-feu
  5. Amélioration des règles du pare-feu
  

