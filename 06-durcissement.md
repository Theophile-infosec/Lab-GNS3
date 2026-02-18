# Durcissement des systèmes Linux et Windows<br>
<br>

**Sommaire** : <br>

I. [Durcissement du serveur Linux](#i-durcissement-du-serveur-linux) <br>
II. [Durcissement des clients](#ii-durcissement-des-clients) <br>

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
<br>
<br>

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

#### a. Identification du fichier utilisé

Commande : <br>
*ps aux | grep dhcpd*

**Objectif** :
- Confirmer le fichier de configuration.<br>
<br>
Capture d'écran : <br>
<br>
<img width="514" height="39" alt="image" src="https://github.com/user-attachments/assets/214136ae-f63d-4763-a7ba-58c314ec03b2" />
<br>
<br>
<br>

#### b. Restriction des permissions

Commandes : <br>
*sudo chown root:root /etc/dhcp/dhcpd.conf<br>
sudo chmod 640 /etc/dhcp/dhcpd.conf*

**Objectif** :
- Limiter lecture/écriture du fichier DHCP aux processus ou utilisateurs disposant de privilèges root.<br>
<br>
Capture d'écran : <br>
<br>
<img width="527" height="38" alt="image" src="https://github.com/user-attachments/assets/b471bcb3-7cc4-45d1-92fd-f51a757233b0" />
<br>
<br>
<br>

## II. Durcissement des clients

### 1. Durcissement client Linux

#### a. Vérification des comptes

Commandes : <br>
*cat /etc/passwd<br>
groups user-lab*

**Objectif** :
- Identifier les privilèges.<br>
<br>
Capture d'écran : <br>
<br>
<img width="325" height="33" alt="image" src="https://github.com/user-attachments/assets/a0b9cf52-3cc0-49e3-b6a3-08c216f568ae" />
<img width="550" height="31" alt="image" src="https://github.com/user-attachments/assets/e3056ae0-b419-49a5-968e-3334f81a97ee" />
<img width="663" height="44" alt="image" src="https://github.com/user-attachments/assets/39906d2a-ff4f-4edc-82e9-cb709e135a75" />
<br>
<br>
<br>

#### b. Création d’un compte administrateur dédié

Commandes : <br>
*sudo adduser admin-lab<br>
sudo usermod -aG sudo admin-lab*

**Objectif** :
- Séparer compte administrateur et compte utilisateur.<br>
<br>
Capture d'écran : <br>
<br>
<img width="816" height="68" alt="image" src="https://github.com/user-attachments/assets/c671f4f8-0606-4151-97e5-5dc1881a680c" />
<br>
<br>
<br>

#### c. Retrait des privilèges sudo au compte utilisateur

Commandes : <br>
*sudo deluser user-lab sudo*

**Objectif** :
- Supprimer toute élévation permanente pour l’utilisateur standard.<br>
<br>
Capture d'écran : <br>
<br>
<img width="661" height="89" alt="image" src="https://github.com/user-attachments/assets/748f5ef1-be01-484d-a89f-14fad9230307" />
<br>
<br>
<br>

#### d. Sécurisation du répertoire personnel

Commandes : <br>
*chmod 700 /home/user-lab*

**Objectif** :
- Empêcher la lecture du home par d’autres utilisateurs.<br>
<br>
Capture d'écran : <br>
<br>
<img width="764" height="71" alt="image" src="https://github.com/user-attachments/assets/0be82186-ed5f-42dd-95ee-b4fc9d46f1d4" />
<img width="828" height="114" alt="image" src="https://github.com/user-attachments/assets/ec84f0b2-ffe9-49f1-a837-77461609f067" />
<br>
<br>
<br>

#### e. Vérification SSH client

Commandes : <br>
*systemctl status ssh*

**Objectif** :
- Confirmer qu’aucun service SSH inutile n’est exposé.<br>
<br>
Capture d'écran : <br>
<br>
<img width="819" height="52" alt="image" src="https://github.com/user-attachments/assets/90eabe6a-53d1-42ae-bb41-8d2eb5cf9e4b" />
<br>
<br>
Conclusion client Linux<br>
- Séparation admin/utilisateur<br>
- Pas de sudo permanent<br>
- Home isolé<br>
- Surface d’exposition réduite<br>
<br>
<br>
<br>

### 2. Durcissement client Windows

#### a. Vérification des comptes

Commandes : <br>
*net user<br>
whoami /groups*

**Objectif** :
- Identifier les comptes administrateurs.<br>
<br>
Capture d'écran : <br>
<br>
<img width="467" height="121" alt="image" src="https://github.com/user-attachments/assets/5f7714ec-19cf-408d-b81e-0ab6c42fcea4" />
<img width="250" height="31" alt="image" src="https://github.com/user-attachments/assets/094609b9-b5ac-41ed-b9fc-b5792c378965" />
<br>
<br>

#### b. Création d’un compte admin dédié

Commandes : <br>
*net user admin.lab MotDePasse /add<br>
net localgroup Administrateurs admin.lab /add*

**Objectif** :
- Séparer administration et utilisation standard.<br>
<br>
Capture d'écran : <br>
<br>
<img width="435" height="197" alt="image" src="https://github.com/user-attachments/assets/99961b9d-a91c-4e0a-9533-bc65b468e0f2" />
<br>
<br>

#### c. Retrait du compte user.lab du groupe Administrateurs

Commandes : <br>
*net localgroup Administrateurs user.lab /delete*

**Objectif** :
- Supprimer les privilèges administrateur a l'utilisateur standard.<br>
<br>
Capture d'écran : <br>
<br>
<img width="558" height="247" alt="image" src="https://github.com/user-attachments/assets/eace1103-8b38-45df-8ad2-e61bfe538c7c" />
<br>
<br>

#### d. Vérification UAC (User Account Control)

**Objectif** :
- Valider la demande d’identifiants administrateur depuis user.lab.<br>
<br>
Capture d'écran : <br>
<br>
<img width="449" height="481" alt="image" src="https://github.com/user-attachments/assets/13b59e2f-35aa-414b-8c4b-5c327e8f28f0" />
<br>
<br>

#### e. Vérification absence SSH

Commandes : <br>
*sc query sshd*

**Objectif** :
- Confirmer qu’aucun service SSH n’est exposé.<br>
<br>
Capture d'écran : <br>
<br>
<img width="501" height="74" alt="image" src="https://github.com/user-attachments/assets/c486ff51-e1ce-4df0-8acb-9e59c16f09f2" />
<br>
<br>

# Conclusion :<br>
<br>
Le durcissement mis en place :
- Supprime les privilèges excessifs
- Applique le principe du moindre privilège
- Isole les comptes
- Réduit l’exposition réseau locale

La surface d’attaque est volontairement réduite tout en conservant un environnement exploitable pour les scénarios offensifs ultérieurs.
