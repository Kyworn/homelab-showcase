# Architecture de Virtualisation

Ce document présente l'organisation des machines virtuelles (VMs) et des conteneurs (LXC) sur l'hyperviseur Proxmox VE. L'utilisation majoritaire de conteneurs LXC vise à optimiser l'utilisation des ressources système.

## Diagramme de Virtualisation

```mermaid
graph TD
    subgraph "Serveur Physique"
        PVE[Proxmox VE Host]
    end

    subgraph "Stockage Externe"
        TRUENAS[<-- Stockage NFS/SMB<br/>(TrueNAS 'Tank')]
    end

    PVE -- Monte le stockage --> TRUENAS

    subgraph "Virtualisation sur Proxmox"
        subgraph "Conteneurs LXC"
            direction LR
            
            subgraph "Stack Média"
                direction TB
                LXC_SONARR[Sonarr (CT 109)]
                LXC_RADARR[Radarr (CT 108)]
                LXC_PROWLARR[Prowlarr (CT 111)]
                LXC_BAZARR[Bazarr (CT 103)]
                LXC_QBIT[qBittorrent (CT 104)]
                LXC_JELLYSEERR[Jellyseerr (CT 119)]
            end

            subgraph "Monitoring & Métriques"
                direction TB
                LXC_UPTIME[Uptime Kuma (CT 101)]
                LXC_GRAFANA[Grafana (CT 115)]
                LXC_INFLUXDB[InfluxDB (CT 117)]
            end

            subgraph "Services Coeur & Projets"
                direction TB
                LXC_DOCKER[Docker (CT 112)]
                LXC_GITEA[Gitea (CT 120)]
                LXC_N8N[n8n (CT 124)]
                LXC_TRANSIT[transit-tcl (CT 121)]
                LXC_MYSQL[MySQL (CT 116)]
            end

            subgraph "Passerelles Réseau"
                direction TB
                LXC_NPM[Nginx Proxy Manager (CT 118)]
                LXC_CLOUDFLARED[cloudflared (CT 110)]
            end
            
            subgraph "Domotique"
                direction TB
                LXC_HOMEBRIDGE[Homebridge (CT 102)]
            end
        end

        subgraph "Machines Virtuelles"
            VM_HAOS[Home Assistant OS (VM 114)]
        end
    end

    PVE -- Héberge --> LXC_SONARR
    PVE -- Héberge --> LXC_RADARR
    PVE -- Héberge --> LXC_PROWLARR
    PVE -- Héberge --> LXC_BAZARR
    PVE -- Héberge --> LXC_QBIT
    PVE -- Héberge --> LXC_JELLYSEERR
    PVE -- Héberge --> LXC_UPTIME
    PVE -- Héberge --> LXC_GRAFANA
    PVE -- Héberge --> LXC_INFLUXDB
    PVE -- Héberge --> LXC_DOCKER
    PVE -- Héberge --> LXC_GITEA
    PVE -- Héberge --> LXC_N8N
    PVE -- Héberge --> LXC_TRANSIT
    PVE -- Héberge --> LXC_MYSQL
    PVE -- Héberge --> LXC_NPM
    PVE -- Héberge --> LXC_CLOUDFLARED
    PVE -- Héberge --> LXC_HOMEBRIDGE
    PVE -- Héberge --> VM_HAOS

```

## Stratégie de Virtualisation

*   **Conteneurs LXC** : La majorité des services sont déployés dans des conteneurs Linux pour leur faible surcharge et leur agilité. Chaque service est isolé dans son propre conteneur pour faciliter la gestion, les mises à jour et la sécurité.
*   **Machines Virtuelles (VM)** : Les VMs sont réservées aux applications qui nécessitent un OS complet ou qui ne sont pas compatibles avec la conteneurisation, comme c'est le cas pour **Home Assistant OS**.
