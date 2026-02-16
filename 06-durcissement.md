# (EN COURS) Durcissement des systèmes Linux et Windows<br>
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

Infos :
- *UID 0* → compte root
- *Shell /usr/sbin/nologin* → compte système
- *Shell /bin/bash* → compte utilisateurs<br>
<br>
Capture d'écran : <br>
<br>
<img width="419" height="52" alt="image" src="https://github.com/user-attachments/assets/15aedb6e-4189-4943-88b7-b5ec28e71fcf" /><br>
<br>
<br>

#### b. Vérifier que sysadmin est administrateur

Commande : <br>
*groups sysadmin*

**Objectif** :
- Vérifier l’appartenance au groupe sudo.<br>

Infos :
- *sudo* → élévation de privilèges autorisée
- *adm* → lecture de logs
- *lxd* → gestion conteneurs<br>
<br>
Capture d'écran : <br>
<br>
<img width="420" height="34" alt="image" src="https://github.com/user-attachments/assets/1c9abd00-c50b-45fd-943b-9f1f6f8c6c1c" /><br>
<br>
<br>

### 2. Sécurisation SSH

#### a. Vérification de la configuration actuelle

Commande : <br>
*sudo grep PermitRootLogin /etc/ssh/sshd_config*

**Objectif** :
- Contrôler la politique d’accès SSH pour le compte root.<br>

Infos :
- *PermitRootLogin prohibit-password* → root autorisé uniquement par clé publique.<br>
<br>
Capture d'écran : <br>
<br>
<img width="556" height="69" alt="image" src="https://github.com/user-attachments/assets/cae1767f-c6cf-4b6b-81dc-5a774f969e6a" /><br>
<br>
<br>

#### b. Désactivation complète de l’accès root

Modification : <br>
*PermitRootLogin no*

Commande : <br>
*sudo systemctl restart ssh*

**Objectif** :
- Empêcher toute connexion directe au compte root.<br>
<br>
Capture d'écran : <br>
<br>
<img width="558" height="66" alt="image" src="https://github.com/user-attachments/assets/9f0c74a1-1205-462e-ac9f-4a35665a7ba5" />
<br>
<br>

#### c. Restriction aux comptes autorisés

Fichier :<br>
*/etc/ssh/sshd_config*

Modification : <br>
*AllowUsers sysadmin*

**Objectif** :
- Limiter les connexions SSH à un compte nominatif unique.

Commande : <br>
*sudo systemctl restart ssh*

### 3. Sécurisation des fichiers critiques DNS

#### a. Identifier les fichiers sensibles

Commande : <br>
*ls -l /etc/bind/*

**Objectif** :
- Identifier les fichiers de zone et fichiers de configuration.<br>
<br>
Capture d'écran : <br>
<br>
<img width="529" height="275" alt="image" src="https://github.com/user-attachments/assets/adaf659d-1334-48fe-ae78-29d743df5ad8" /><br>
<br>
<br>

#### b. Réduction des permissions

Commandes : <br>
*sudo chmod 640 /etc/bind/db.lab.local<br>
sudo chmod 640 /etc/bind/db.10.10.10.30<br>
sudo chmod 640 /etc/bind/named.conf.local<br>
sudo chmod 640 /etc/bind/named.conf.options<br>*

**Objectif** :
Appliquer le principe du moindre privilège :
- *root* → lecture/écriture
- *bind* → lecture
- *others* → aucun accès<br>
<br>
Capture d'écran : <br>
<br>
<img width="615" height="209" alt="image" src="https://github.com/user-attachments/assets/61d95424-d3ae-45a7-b8ce-acde1d16997a" /><br>
<br>
<br>

### 4. Sécurisation des fichiers DHCP
