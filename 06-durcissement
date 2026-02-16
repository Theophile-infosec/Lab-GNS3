# Durcissement des systèmes Linux et Windows<br>
<br>

**Sommaire** : <br>

I.  <br>
II.  <br>

<br>

# Objectifs :<br>
<br>

Mettre en place un durcissement de base des systèmes Linux et Windows afin de :
- Séparer les comptes administrateurs et utilisateurs standards
- Supprimer les accès privilégiés inutiles
- Restreindre les accès SSH
- Appliquer le principe du moindre privilège sur les fichiers critiques
- Réduire la surface d’attaque des postes clients

**Topologie du laboratoire :**
<br>
<br>
<img width="611" height="453" alt="image" src="https://github.com/user-attachments/assets/d53416bd-ed0a-46c9-a8d2-e2302ebf31b9" />
<br>

- Serveur DHCP : Linux_Server-1 (e0)
- Clients : Linux_Desktop-1 (e0), Windows-10_Desktop-1 (e0)
- Commutation : Linux_Switch
- Pare-feu : Linux_Fierwall (e0)
- Plan d'adressage : VLAN10 → 10.10.10.0/24, VLAN20 → 10.10.20.0/24, VLAN30 → 10.10.30.0/24<br>
<br>

# Procédures :<br>
<br>

## I. Durcissement du serveur Linux

### 1. Gestion des comptes

#### a. Vérification des comptes existants

Commande : <br>
*cat /etc/passwd*

**Objectif** :
- Identifier les comptes système et les comptes utilisateurs.<br>

Format : 
*login:password:UID:GID:GECOS:home:shell*

Points de contrôle :
- *UID 0* → compte root
- *Shell /usr/sbin/nologin* → compte système
- *Shell /bin/bash* → compte utilisateurs<br>
<br>
Capture d'écran : <br>
<br>
<br>
<br>
<br>
