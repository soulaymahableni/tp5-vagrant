# TP5 — Environnements de lab reproductibles avec Vagrant

Vagrant décrit les machines virtuelles dans un `Vagrantfile` et demande à VirtualBox de les construire ;
Ansible configure ensuite ces VM en réutilisant les rôles du TP4.

## Environnement

| Élément | Valeur |
|---|---|
| Hôte physique | Windows, VMware Workstation, 20 Go de RAM |
| VM de lab | `vagrant-lab`, Ubuntu 26.04, IP 192.168.190.157, 4 vCPU, 58 Go de disque, VT-x exposé |
| Outils dans la VM de lab | VirtualBox, Vagrant 2.4.9, Ansible |
| Box | `bento/ubuntu-24.04` |

Virtualisation imbriquée : VMware (hôte Windows) → VirtualBox (dans `vagrant-lab`) → VM Vagrant.
Sur Windows, Hyper-V et l'intégrité de la mémoire ont été désactivés pour que VMware expose VT-x à la VM de lab.

```bash
grep -Ec '(vmx|svm)' /proc/cpuinfo    # 4
vagrant --version                     # Vagrant 2.4.9
```

![Virtualisation imbriquée](captures/nested-virt.png)

## Structure

```
vagrant_demo/
├── Vagrantfile          provision shell (Apache), dossier synchronisé web/ → /var/www/html, port 80 → 8080
└── web/index.html
vagrant_lab/
├── Vagrantfile          2 VM (lab1, lab2) en private_network + provisioner ansible
├── playbook.yml         appelle les rôles du TP4
├── docker-compose.yml   conteneur web (port 8082)
└── roles/               jenkins_install, docker_portainer_install, container_deployment
```

## Partie 1 — Démo Vagrant (`vagrant_demo/`)

```bash
vagrant box add bento/ubuntu-24.04 --provider virtualbox
vagrant up
curl http://localhost:8080          # depuis la VM de lab
vagrant ssh -c "hostname"           # vagrant-web1
vagrant destroy -f
```

Chemin d'une requête : navigateur du PC → `192.168.190.157:8080` (VM de lab) → port 80 de l'invitée → Apache qui sert
le dossier `web/` synchronisé.

![Démo Vagrant](captures/vagrant-demo.png)

## Partie 2 — Lab de 2 VM provisionnées par Ansible (`vagrant_lab/`)

| VM | IP privée | Jenkins | Portainer | Site web |
|---|---|---|---|---|
| lab1 | 192.168.56.11 | :18080 | :19443 | :18082 |
| lab2 | 192.168.56.12 | :28080 | :29443 | :28082 |

Les ports des VM sont redirigés sur l'IP de la VM de lab (`host_ip: "0.0.0.0"`), pour être joignables depuis le PC :
`http://192.168.190.157:18082`, `https://192.168.190.157:19443`, etc.

```bash
vagrant validate
vagrant up
vagrant status
```

![Vagrantfile du lab](captures/lab-vagrantfile.png)

![vagrant up](captures/lab-up.png)

Provisionnement Ansible de `lab2` terminé (`failed=0`) :

![Provisionnement lab2](captures/lab2-provision.png)

Les deux VM en fonctionnement :

![vagrant status](captures/vagrant-status.png)

Services de `lab1` testés à travers les ports redirigés (site 200, Jenkins 200, Portainer 307 = redirection vers la page
de connexion), ports en écoute sur `0.0.0.0` :

![Services du lab](captures/lab-services.png)

## Commandes Vagrant utilisées

| Commande | Rôle |
|---|---|
| `vagrant box add` | télécharge et met en cache une box |
| `vagrant up` / `vagrant up lab2 --provision` | crée et démarre les VM / force le provisionnement |
| `vagrant provision lab2` | relance Ansible sur une VM en marche |
| `vagrant status` | état des VM du projet |
| `vagrant ssh lab2 -c "..."` | exécute une commande dans une VM |
| `vagrant halt -f` / `vagrant destroy -f` | arrête / supprime les VM |

## Rapport

Voir [RAPPORT.md](RAPPORT.md).
