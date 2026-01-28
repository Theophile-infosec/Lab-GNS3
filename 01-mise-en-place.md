# Mise en place de l’environnement du laboratoire<br>
<br>

# Objectifs :<br>
<br>

Ce chapitre a pour objectif de mettre en place l’environnement de travail du laboratoire GNS3.

Il consiste à :

- installer les différentes machines virtuelles nécessaires ;
- préparer les templates dans GNS3.

Aucune configuration réseau (adressage IP, DHCP, VLAN, routage ou pare-feu) n’est réalisée à ce stade.

# Procédures :<br>
<br>

## I. Environnement technique

Le laboratoire est réalisé avec :
  - GNS3 (version : 2.2.55)
  - VMware Workstation (version : 17.6.4)
  - Système hôte : Windows 11

Les machines suivantes sont intégrées au projet :
  - 1 serveur Linux – Ubuntu Server (version : 24.04.3 LTS)
  - 1 client Linux – Ubuntu Desktop (version : 24.04.3 LTS)
  - 1 client Windows – Windows 10 (version : 22H2)
  - 1 machine de test sécurité – Kali Linux (version : 2025.4)

Les ressources allouées sont les suivantes :
  - Ubuntu Server : 2 vCPU / 2 Go RAM
  - Ubuntu Desktop : 3 vCPU / 4 Go RAM
  - Windows 10 : 3 vCPU / 4 Go RAM
  - Kali Linux : 3 vCPU / 4 Go RAM

## II. Création des templates

Chaque système a été importé dans GNS3 sous forme de template distinct :
  - Template Ubuntu Server
  - Template Ubuntu Desktop
  - Template Windows 10
  - Template Kali Linux

La création des templates permet :
  - la standardisation des déploiements ;
  - la reproductibilité du laboratoire ;
  - la duplication rapide d’instances si nécessaire.


# Conclusion :<br>
<br>

À l’issue de cette étape :

  - L’environnement de virtualisation est opérationnel ;
  - Les templates sont fonctionnels ;
  - Le projet est structuré et prêt à recevoir les configurations réseau.
