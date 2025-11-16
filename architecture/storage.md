# Architecture de Stockage

Le stockage du homelab est centralisé sur un serveur dédié tournant sous TrueNAS CORE, utilisant le système de fichiers ZFS pour sa résilience et ses fonctionnalités avancées.

## Diagramme de Stockage (TrueNAS)

```mermaid
graph TD
    subgraph "Serveur de Stockage"
        TN[TrueNAS]
    end

    subgraph "Pool ZFS"
        POOL[Pool 'Tank']
    end
    
    subgraph "Topologie du Pool (RAID 1)"
        MIRROR[Mirror-0]
    end

    subgraph "Disques Physiques"
        DISK1[Disk 1 (sda)]
        DISK2[Disk 2 (sdb)]
    end

    subgraph "Clients du Stockage"
        PVE[Proxmox VE]
        SERVICES[Autres services<br/>(ex: *Arr stack)]
    end

    subgraph "Partages Réseau"
        NFS[Partages NFS<br/>(pour Proxmox)]
        SMB[Partages SMB<br/>(pour les clients)]
    end
    
    subgraph "Datasets ZFS"
        DS_VMS[Dataset pour VMs & CTs]
        DS_MEDIA[Dataset Média]
        DS_BACKUPS[Dataset Backups]
        DS_AUTRES[...]
    end

    TN -- Gère --> POOL
    POOL -- Contient --> DS_VMS
    POOL -- Contient --> DS_MEDIA
    POOL -- Contient --> DS_BACKUPS
    POOL -- Contient --> DS_AUTRES
    
    POOL -- Est composé de --> MIRROR
    MIRROR -- Contient --> DISK1
    MIRROR -- Contient --> DISK2

    DS_VMS -- Partagé via --> NFS
    DS_MEDIA -- Partagé via --> SMB
    
    NFS -- Monté par --> PVE
    SMB -- Accédé par --> SERVICES
```

## Stratégie de Stockage

*   **Serveur Centralisé** : Un serveur TrueNAS unique gère tout le stockage, ce qui simplifie la gestion, la sauvegarde et la protection des données.
*   **ZFS et Redondance** : Le pool principal `Tank` est configuré en miroir (équivalent RAID 1). Toutes les données écrites sur le pool sont dupliquées sur deux disques physiques. Cela garantit qu'aucune donnée n'est perdue en cas de défaillance d'un des deux disques.
*   **Datasets** : Le pool est organisé en plusieurs datasets ZFS. Chaque dataset peut avoir ses propres règles de snapshots, de quotas et de permissions. C'est utilisé pour séparer logiquement les différents types de données (VMs, médias, backups, etc.).
*   **Partages Réseau** :
    *   **NFS** est principalement utilisé pour fournir du stockage à l'hyperviseur Proxmox. C'est là que les disques virtuels des VMs et des conteneurs sont stockés.
    *   **SMB** (Samba/CIFS) est utilisé pour les partages de fichiers plus classiques, accessibles par les services (comme Sonarr/Radarr pour les médias) ou directement par les utilisateurs sur le réseau local.
*   **Snapshots** : Des tâches de snapshots ZFS sont probablement configurées pour créer des instantanés réguliers des datasets critiques, permettant une restauration rapide en cas d'erreur humaine ou de corruption de données.
