# Rapport — TP5 : Vagrant

## 1. Démarche
Une VM Ubuntu 26.04 (`vagrant-lab`) a été créée dans VMware avec l'option « Virtualiser Intel VT-x/EPT » pour permettre
la virtualisation imbriquée (`grep -Ec '(vmx|svm)' /proc/cpuinfo` = 4). VirtualBox, Ansible et Vagrant 2.4.9 y ont été
installés.

La **démo** (`vagrant_demo/`) crée une VM `vagrant-web1` provisionnée par un script shell (Apache), avec le dossier
`web/` synchronisé dans `/var/www/html` et le port 80 redirigé vers 8080. `curl http://localhost:8080` renvoie
« Bonjour depuis Vagrant ».

Le **lab** (`vagrant_lab/`) est un Vagrantfile multi-machines : une boucle sur `LAB_VMS` définit `lab1` et `lab2`, chacune
sur le réseau privé `192.168.56.0/24`, avec trois ports redirigés (Jenkins, Portainer, site) et le provisioner `ansible`
qui exécute `playbook.yml`. Ce playbook réutilise sans les modifier les trois rôles du TP4 ; seule la variable
`docker_user` est fixée à `vagrant`.

## 2. Résultats
- `lab1` et `lab2` provisionnées avec `failed=0` (Jenkins, Docker, Portainer, conteneur web).
- Services de `lab1` vérifiés via les ports redirigés : site HTTP 200, Jenkins HTTP 200, Portainer 307 (redirection
  vers la page de connexion).
- Depuis Windows, `Test-NetConnection 192.168.190.157 -Port 18082` : `TcpTestSucceeded : True`.

## 3. Intérêt de Vagrant + Ansible
Vagrant rend la **création** des VM reproductible (un seul fichier versionné, `vagrant up` / `destroy`), Ansible rend
leur **configuration** reproductible. Les rôles écrits au TP4 pour un serveur existant ont été réutilisés tels quels
sur des VM neuves : c'est l'intérêt de structurer le code en rôles.

## 4. Difficultés rencontrées
- **Virtualisation imbriquée désactivée** (`vmx/svm` = 0) : il a fallu cocher VT-x dans VMware, désactiver Hyper-V
  (`bcdedit /set hypervisorlaunchtype off`, en PowerShell administrateur) et l'intégrité de la mémoire de Windows,
  puis redémarrer le PC.
- **Disque à moitié utilisé** par l'installeur Ubuntu (29 Go sur 60) : `lvextend -l +100%FREE` puis `resize2fs`.
- **Clé du dépôt HashiCorp absente** (`NO_PUBKEY`) : réinstallation de la clé avec `curl … | gpg --dearmor --yes`.
- **Clé Jenkins** : le dépôt utilise désormais `jenkins.io-2026.key` ; l'ancienne clé de 2023 du rôle TP4 a été remplacée,
  sinon l'installation échoue sur une VM neuve.
- **Manque de mémoire** : avec 7,8 Go pour la VM de lab et 2 × 2,5 Go pour les VM du lab, VirtualBox a mis `lab2` en
  pause pendant l'installation de Docker, puis `lab1` a été arrêtée (`aborted`). Corrections : mémoire des VM ramenée à
  2 Go, `dpkg --configure -a` dans `lab2` pour terminer l'installation interrompue, `vagrant provision lab2`, et RAM de
  la VM de lab augmentée dans VMware.
- **Accès depuis le PC** : les IP privées `192.168.56.x` ne sont visibles que depuis la VM de lab ; l'accès depuis
  Windows se fait par les ports redirigés sur `192.168.190.157`.
- Avertissements sans conséquence : « pull access denied for tp4-webapp » (Compose construit l'image localement) et
  « buildx isn't installed » (Compose utilise le builder classique).

![Incident mémoire](captures/incident-ram.png)
