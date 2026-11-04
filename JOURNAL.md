# Journal de bord

## 2026-11-03 – Tailscale dans un conteneur LXC
- Objectif : accéder à Jellyfin à distance sans ouvrir de port
- Problème : le service tailscaled ne démarrait pas
- Cause : le conteneur non privilégié n'avait pas accès à /dev/net/tun
- Solution : autorisation du périphérique dans /etc/pve/lxc/100.conf, puis redémarrage
- Appris : fonctionnement des conteneurs non privilégiés et des fichiers .conf de Proxmox
