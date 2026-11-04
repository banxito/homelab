# Journal de bord

Suivi chronologique du homelab : ce qui a été fait, les problèmes rencontrés et ce que j'en ai appris.

---

## Septembre 2026 – Premier serveur sous Ubuntu Server
- Objectif : réutiliser un PC portable inutilisé (Lenovo Legion Y520) pour héberger un serveur multimédia
- Réalisé : installation d'Ubuntu Server, puis de CasaOS pour déployer Jellyfin en quelques clics
- Limite : CasaOS masque ce qui se passe en dessous (conteneurs, ports, droits), je ne maîtrisais pas mon installation
- Décision : passer à Proxmox VE pour apprendre la virtualisation et tout configurer moi-même

## [date] – Installation de Proxmox VE
- Problème : la virtualisation était désactivée dans le BIOS (« Intel Virtual Technology : Disabled »)
- Solution : activation de VT-x dans le BIOS, puis vérification avant l'installation
- Appris : un hyperviseur a besoin des extensions de virtualisation du processeur pour lancer des VM

## 2026-11-01 – Stockage : un seul disque
- Constat : Proxmox ne détecte qu'un SSD de 500 Go, qui contient déjà le système (partitions de boot + LVM)
- Erreur évitée : formater la partition LVM aurait effacé Proxmox
- Solution : les films sont stockés sur un volume dédié créé dans local-lvm, monté dans le conteneur sur /media
- Avantage : le volume est séparé du système, exclu des sauvegardes, et pourra être migré vers un disque dédié plus tard

## 2026-11-01 – Conteneur CT100 et Jellyfin
- Création d'un conteneur LXC Debian 12 **non privilégié** (2 cœurs, 2 Go de RAM), avec l'option Nesting pour Docker
- Installation de Docker, puis de Jellyfin via un fichier docker-compose.yml
- Médiathèque montée en lecture seule (`/media:/media:ro`) : Jellyfin peut lire les films mais pas les modifier (moindre privilège)
- Redémarrage automatique : `restart: unless-stopped` côté Docker, « Start at boot » côté Proxmox
- Snapshot pris une fois la configuration fonctionnelle

## 2026-10-29 – Transfert de fichiers
- Création d'un utilisateur dédié et attribution des dossiers de /media avec `chown`
- Erreur `Permission denied` sur `lost+found` : normal, ce dossier appartient au root de l'hôte, inaccessible depuis un conteneur non privilégié. Preuve que l'isolation fonctionne
- SFTP avec FileZilla, après vérification de l'empreinte de la clé du serveur (protection contre l'homme du milieu)
- Mise en place d'un partage Samba pour accéder aux fichiers directement depuis le Finder
- Connexion SSH depuis le Mac avec un raccourci dans `~/.ssh/config`

## 2026-10-29 – Accès distant avec Tailscale
- Objectif : accéder à Jellyfin et aux fichiers depuis l'extérieur sans ouvrir de port sur la box
- Problème : le service `tailscaled` ne démarrait pas dans le conteneur
- Cause : un conteneur non privilégié n'a pas accès à `/dev/net/tun` par défaut
- Solution : autorisation du périphérique dans `/etc/pve/lxc/100.conf`, puis redémarrage du conteneur
- Piège identifié : dans ce fichier, tout ce qui est écrit sous une section de snapshot (`[nom]`) ne s'applique qu'au snapshot
- Réglages : expiration de la clé désactivée pour le serveur, test réussi depuis un téléphone en 4G

## 2026-10-29 – Optimisation matérielle
- Batterie débranchée : supprime le risque d'usure et de gonflement d'une batterie chargée en permanence
- Contrepartie : plus de protection contre les coupures de courant, d'où l'importance des sauvegardes
- Extinction automatique de l'écran avec le paramètre `consoleblank=60` dans GRUB, pour réduire la consommation

## 2026-11-02 – Accès à Proxmox à distance par tunnel SSH
- Problème : l'interface Proxmox est inaccessible à distance, car Tailscale n'est installé que dans CT100
- Solution temporaire : redirection de port SSH via le conteneur (`ssh -L 8006:<IP-proxmox>:8006`)
- Deuxième problème : la connexion restait bloquée, car le raccourci SSH pointait vers l'adresse locale du conteneur
- Solution : raccourci modifié pour utiliser le nom Tailscale, valable à la maison comme à l'extérieur
- Appris : redirection de port SSH, différence entre adresse locale et adresse Tailscale

## 2026-11-04 – Incident : perte de l'accès SSH
- Action : désactivation de l'authentification par mot de passe dans la configuration SSH du conteneur
- Problème : aucune clé SSH n'avait été copiée sur le serveur, la connexion est devenue impossible
- Conséquence : plus d'accès à distance au conteneur, ni à Proxmox (le tunnel passait par ce même SSH)
- Correction prévue : suppression du fichier de durcissement depuis la console Proxmox, copie de la clé, puis nouvelle tentative
- Leçons retenues :
  - toujours vérifier qu'une méthode d'accès fonctionne avant d'en couper une autre
  - garder une session ouverte pendant une modification et tester dans une nouvelle fenêtre
  - prévoir deux accès distants indépendants (Tailscale sur Proxmox en plus du conteneur)

## 2026-11-04 – Documentation sur GitHub
- Création d'une clé SSH Ed25519 sur le Mac et ajout sur GitHub comme clé d'authentification
- Création de ce dépôt pour documenter l'architecture, les choix techniques et les incidents
- Règle : aucun secret publié (mots de passe, clés privées, clés Tailscale)

---

## Prochaines étapes
- [ ] Rétablir l'accès SSH, copier la clé et réappliquer le durcissement (clés uniquement, root interdit, fail2ban)
- [ ] Installer Tailscale sur le nœud Proxmox pour un second accès distant
- [ ] Activer la double authentification sur l'interface Proxmox
- [ ] Planifier des sauvegardes automatiques des conteneurs
- [ ] Serveur Minecraft dans un conteneur dédié
- [ ] Labo de pentest isolé (Kali Linux, Metasploitable)
- [ ] Supervision de sécurité avec Waz
