# Automation : Sauvegardes

Cette section détaille la stratégie de sauvegarde mise en place pour prévenir la perte de données.

## 1. Sauvegardes Proxmox VE

*   **Fréquence** : Hebdomadaire, le dimanche à 01:00.
*   **Cible** : Les sauvegardes des conteneurs sont stockées sur le stockage Proxmox nommé "Backup".
*   **Rétention** : Conservation des 2 dernières sauvegardes.
*   **Compression** : zstd.
*   **Services Inclus** : Conteneurs LXC 101, 102, 104, 106, 108, 109, 110, 111, 112, 115, 116, 117.

## 2. Snapshots TrueNAS (ZFS)

Les snapshots ZFS sont utilisés pour une protection instantanée et une restauration rapide des données.

*   **Fréquence** : Quotidienne, à 00:00.
*   **Datasets Concernés** :
    *   `Tank/share` (non récursif)
    *   `Tank/ix-applications` (récursif)
    *   `Tank/server/project` (récursif)
    *   `Tank/server/download` (non récursif)
    *   `Tank/server/backup` (non récursif)
*   **Rétention** : Conservation des snapshots pendant 14 jours.

## 3. Tâches de Réplication

*   **Description** : Aucune tâche de réplication n'est configurée pour le moment.
