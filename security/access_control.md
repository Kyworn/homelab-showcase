# Sécurité : Contrôle d'Accès

Cette section décrit les différentes couches de contrôle d'accès pour sécuriser les services du homelab.

## 1. Accès Externe (Cloudflare Zero Trust)

L'accès aux services exposés sur Internet est protégé par la suite Zero Trust de Cloudflare.

*   **Politiques d'Accès** : Les services exposés via le tunnel `zserv` sont protégés par des politiques d'accès Cloudflare (authentification, règles géographiques, etc.).
*   **Services Protégés** : Les services exposés via le tunnel `zserv` (ex: Gitea, certains dashboards) sont protégés par Cloudflare Access.

## 2. Accès Interne (Nginx Proxy Manager)

L'accès aux services sur le réseau local est géré par NPM.

*   **Listes d'Accès** : Nginx Proxy Manager est utilisé uniquement pour l'accès local, aucune liste d'accès externe n'est configurée.
*   **Authentification HTTP** : Non utilisée par défaut, l'accès est géré par les applications elles-mêmes.

## 3. Accès aux Serveurs (SSH)

L'accès direct aux serveurs (Proxmox, TrueNAS) et aux conteneurs est restreint.

*   **Authentification par Clé** : L'accès SSH est configuré pour n'autoriser que l'authentification par clé publique. Les mots de passe sont désactivés.
*   **Utilisateurs** : L'accès SSH sur Proxmox est limité à une clé spécifique.
