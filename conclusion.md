# Conslusion du laboratoire<br>
<br>

**Sommaire** : <br>

I. [Objectifs]() <br>
II. [Architecture]() <br>
III. [Résultats]() <br>
IV. [Surface d'attaque]() <br>
V. [Limites]() <br>
VI. [Suites]() <br>

<br>

## I. Objectifs
<br>
Ce laboratoire avait pour objectif de :<br>
- concevoir une infrastructure réseau virtualisée sous GNS3 intégrant des services essentiels (DHCP, DNS, segmentation VLAN et pare-feu inter-VLAN)<br>
- vérifier la cohérence de la configuration avec des tests fonctionnels
<br>
<br>

## II. Architecture
<br>
L’environnement repose sur plusieurs machines virtuelles réparties en VLAN distincts : clients Linux, clients Windows et serveur. <br>
La topologie est basée sur un switch Linux configuré en bridge avec segmentation VLAN. <br>
Le service DNS interne est hébergé sur le serveur Linux. <br>
Le routage inter-VLAN est assuré par un pare-feu Linux, avec DHCP relay. <br>
<br>
<br>

## III. Résultats
<br>
- Les services DHCP et DNS fonctionnent correctement dans une architecture segmentée. <br>
- La distribution des paramètres réseau est assurée via relay DHCP. <br>
- La résolution DNS directe et inverse est opérationnelle. <br>
- Le pare-feu applique une politique restrictive validée par capture réseau -> seules les flux explicitement autorisés atteignent le serveur.<br>
<br>
<br>

## IV. Surface d'attaque
<br>
- Les tests de reconnaissance active et passive ont montré une surface d’exposition limitée -> un seul service réseau accessible (DNS sur le port 53).<br>
- Aucune exposition des clients.<br>
- Absence de transfert de zone DNS ou de récursion ouverte. <br>
- Les scans ont confirmé le filtrage des ports non autorisés.<br>
<br>
<br>

## V. Limites
<br>
- L’environnement reste volontairement simplifié avec peu d’hôtes et de services applicatifs. <br>
- L’absence d’infrastructure d’identité centralisée (Active Directory) limite les scénarios d’attaque avancés et les tests de mouvements latéraux.
<br>
<br>

## VI. Suites
<br>
Le prochain laboratoire portera sur Active Directory afin d’étudier une architecture plus proche d’un environnement entreprise. 
Il permettra d'étudier et de manipuler la gestion centralisée des identités, les services Windows et des scénarios de sécurité plus avancés.
