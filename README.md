# 🏠 Homelab Infrastructure - Zorko

> Infrastructure auto-hébergée complète avec virtualisation, stockage ZFS, et accès sécurisé via Cloudflare Zero Trust

![Proxmox](https://img.shields.io/badge/Proxmox-VE_9.0.6-E57000?logo=proxmox&logoColor=white)
![TrueNAS](https://img.shields.io/badge/TrueNAS-Scale-0095D5?logo=truenas&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare-Zero_Trust-F38020?logo=cloudflare&logoColor=white)
![ZFS](https://img.shields.io/badge/ZFS-RAID1-00979D?logo=openzfs&logoColor=white)
![LXC](https://img.shields.io/badge/LXC-19_Containers-success)
![Storage](https://img.shields.io/badge/Storage-79.5%25_Used-orange)

---

## 📊 Vue d'Ensemble de l'Infrastructure

### Statistiques Actuelles

| Métrique | Valeur | Détails |
|----------|--------|---------|
| **Conteneurs LXC** | 19 actifs / 22 total | Production 24/7 |
| **Machines Virtuelles** | 1 (Home Assistant) | Actuellement arrêtée |
| **Capacité Stockage** | 928 GB | 738 GB utilisés (79.5%) |
| **RAM Proxmox** | 30.75 GB | 7 GB utilisés, 24 GB disponibles |
| **CPU Hyperviseur** | AMD Ryzen 7 5800U | 8 cores / 16 threads @ 3068 MHz |
| **Uptime Proxmox** | 5.2 jours | Redémarrage récent |
| **Snapshots ZFS** | 5 tâches quotidiennes | Rétention 14 jours |

### Répartition du Stockage TrueNAS

```mermaid
pie title Utilisation Stockage Tank (738 GB / 928 GB)
    "Séries TV" : 355
    "Films" : 349
    "Backups" : 22.8
    "Applications" : 5.11
    "Downloads" : 3.37
    "Projets Git" : 2.23
    "Espace Libre" : 161
```

---

## 🏗️ Architecture Globale

### Flux de Trafic Internet → Services

```mermaid
graph LR
    A[🌐 Internet] -->|HTTPS| B[☁️ Cloudflare Edge]
    B -->|WAF + DDoS Protection| C{🔐 Access Control}
    C -->|✅ Authentifié| D[🔒 Tunnel Chiffré]
    C -->|❌ Bloqué| E[⛔ Access Denied]
    D -->|8 Connexions HA| F[📡 Cloudflared]
    F -->|Reverse Proxy| G[🔀 Nginx Proxy Manager]
    G -->|Route vers| H[🎯 Services LXC]

    style B fill:#f38020,stroke:#333,stroke-width:2px,color:#fff
    style C fill:#f9a825,stroke:#333,stroke-width:2px
    style D fill:#4caf50,stroke:#333,stroke-width:2px,color:#fff
    style E fill:#f44336,stroke:#333,stroke-width:2px,color:#fff
    style F fill:#2196f3,stroke:#333,stroke-width:2px,color:#fff
    style G fill:#9c27b0,stroke:#333,stroke-width:2px,color:#fff
    style H fill:#00bcd4,stroke:#333,stroke-width:2px,color:#fff
```

### Architecture Infrastructure Complète

```mermaid
graph TB
    subgraph CLOUD["☁️ Cloudflare Zero Trust"]
        DNS[🌍 DNS zorko.xyz]
        TUNNEL[🔒 Tunnel 'zserv']
        WAF[🛡️ WAF + DDoS]
        ACCESS[🔐 Access Apps]
    end

    subgraph EDGE["🏠 Réseau Domestique - FTTH 10G"]
        ROUTER[📡 Freebox v7<br/>10 Gbit/s ↓ / 900 Mbit/s ↑]
    end

    subgraph COMPUTE["💻 Proxmox VE 9.0.6 - Hyperviseur"]
        PVE[⚙️ AMD Ryzen 7 5800U<br/>8C/16T @ 3GHz<br/>30.75 GB RAM]

        subgraph LXC_INFRA["Infrastructure (4 CT)"]
            CT_CLOUD[📡 Cloudflared]
            CT_NPM[🔀 Nginx Proxy<br/>Manager]
            CT_DOCKER[🐋 Docker Host]
            CT_UPTIME[📊 Uptime Kuma]
        end

        subgraph LXC_MEDIA["Stack Média (7 CT)"]
            CT_SONARR[📺 Sonarr]
            CT_RADARR[🎬 Radarr]
            CT_BAZARR[💬 Bazarr]
            CT_PROWLARR[🔍 Prowlarr]
            CT_OVERSEERR[📋 Overseerr]
            CT_JELLYSEERR[📝 Jellyseerr]
            CT_QBIT[⬇️ qBittorrent]
        end

        subgraph LXC_DEV["Dev & Automation (3 CT)"]
            CT_GITEA[🗂️ Gitea Git]
            CT_N8N[⚡ n8n Workflows]
            CT_DASH[📱 Dashboard]
        end

        subgraph LXC_DATA["Monitoring & Data (3 CT)"]
            CT_GRAF[📈 Grafana]
            CT_INFLUX[💾 InfluxDB]
            CT_MYSQL[🗄️ MySQL]
        end

        subgraph LXC_HOME["Domotique (1 CT + 1 VM)"]
            CT_HB[🏡 Homebridge]
            VM_HA[🏠 Home Assistant<br/>⏸️ Stopped]
        end
    end

    subgraph STORAGE["💿 TrueNAS Scale - Stockage ZFS"]
        NAS[🗄️ Pool 'Tank' RAID 1<br/>928 GB Total<br/>738 GB Utilisés]

        subgraph DATASETS["📁 Datasets (9 total)"]
            DS_MEDIA[🎬 Médias: 704 GB<br/>Films + Séries]
            DS_BACKUP[💾 Backups: 22.8 GB]
            DS_GIT[🗂️ Git: 2.23 GB]
            DS_APP[⚙️ Apps: 5.11 GB]
        end

        SNAP[📸 5 Snapshots Daily<br/>Rétention 14 jours]
    end

    DNS -->|Résout| TUNNEL
    WAF -->|Protège| TUNNEL
    ACCESS -->|Auth| TUNNEL
    TUNNEL ==>|Chiffré| ROUTER

    ROUTER ==>|1 Gbit/s| PVE

    PVE --> LXC_INFRA
    PVE --> LXC_MEDIA
    PVE --> LXC_DEV
    PVE --> LXC_DATA
    PVE --> LXC_HOME

    CT_CLOUD -->|Reçoit| CT_NPM
    CT_NPM -->|Route| LXC_MEDIA
    CT_NPM -->|Route| LXC_DEV

    NAS -.NFS/SMB.-> PVE
    NAS -.Médias.-> CT_SONARR
    NAS -.Médias.-> CT_RADARR
    NAS -.Médias.-> CT_QBIT

    NAS --> DATASETS
    NAS --> SNAP

    CT_GRAF -.Métriques.-> CT_INFLUX

    style CLOUD fill:#f38020,stroke:#333,stroke-width:3px,color:#fff
    style EDGE fill:#4caf50,stroke:#333,stroke-width:3px,color:#fff
    style COMPUTE fill:#2196f3,stroke:#333,stroke-width:3px,color:#fff
    style STORAGE fill:#9c27b0,stroke:#333,stroke-width:3px,color:#fff

    style TUNNEL fill:#ffa726,stroke:#333,stroke-width:2px,color:#000
    style NAS fill:#ab47bc,stroke:#333,stroke-width:2px,color:#fff
    style PVE fill:#42a5f5,stroke:#333,stroke-width:2px,color:#fff
```

---

## 🎯 Points Forts Techniques

### Sécurité
- ✅ **Zero-Trust Access** via Cloudflare Tunnel (aucun port ouvert sur Internet)
- ✅ **WAF Cloudflare** avec protection DDoS intégrée
- ✅ **Reverse Proxy** centralisé (Nginx Proxy Manager)
- ✅ **Accès local isolé** via domaines `*.lan`
- ✅ **Snapshots ZFS automatiques** (quotidiens, rétention 14 jours)

### Virtualisation & Infrastructure
- ✅ **19 conteneurs LXC** en production 24/7
- ✅ **Proxmox VE 9.0.6** avec kernel Linux 6.14.11
- ✅ **Ressources optimisées** : allocation dynamique CPU/RAM
- ✅ **Monitoring** : Grafana + InfluxDB pour métriques temps réel
- ✅ **Uptime Monitoring** : Uptime Kuma pour supervision des services

### Stockage & Données
- ✅ **ZFS RAID 1** (miroir) sur 2x 1TB WD Red
- ✅ **Compression LZ4** activée (-20% économie d'espace)
- ✅ **704 GB de médias** (Films + Séries) organisés
- ✅ **Backups Proxmox** : hebdomadaires, compression zstd
- ✅ **Snapshots TrueNAS** : quotidiens, 14 jours de rétention

### Automation & DevOps
- ✅ **Stack *Arr complète** (Sonarr, Radarr, Prowlarr, Bazarr)
- ✅ **n8n** pour workflows d'automatisation
- ✅ **Gitea** auto-hébergé pour projets Git
- ✅ **Infrastructure as Code** : documentation complète sur GitHub

---

## 📁 Services en Production

### Stack Média (Arr Suite)
| Service | Conteneur | Rôle |
|---------|-----------|------|
| **Sonarr** | LXC 109 | Gestion séries TV |
| **Radarr** | LXC 108 | Gestion films |
| **Prowlarr** | LXC 111 | Indexeur torrents |
| **Bazarr** | LXC 103 | Sous-titres automatiques |
| **Overseerr** | LXC 106 | Requêtes médias |
| **Jellyseerr** | LXC 119 | Requêtes médias (fork) |
| **qBittorrent** | LXC 104 | Client torrent |

### Infrastructure & Proxy
| Service | Conteneur | Rôle |
|---------|-----------|------|
| **Nginx Proxy Manager** | LXC 118 | Reverse proxy central |
| **Cloudflared** | LXC 110 | Tunnel Cloudflare |
| **Docker** | LXC 112 | Hôte conteneurs Docker |
| **Uptime Kuma** | LXC 101 | Monitoring uptime |

### Development & Automation
| Service | Conteneur | Rôle |
|---------|-----------|------|
| **Gitea** | LXC 120 | Git auto-hébergé |
| **n8n** | LXC 124 | Workflows automation |
| **Dashboard** | LXC 122 | Dashboard services |

### Monitoring & Databases
| Service | Conteneur | Rôle |
|---------|-----------|------|
| **Grafana** | LXC 115 | Visualisation métriques |
| **InfluxDB** | LXC 117 | Base de données time-series |
| **MySQL** | LXC 116 | Base de données relationnelle |

### Home Automation
| Service | Type | État | Rôle |
|---------|------|------|------|
| **Homebridge** | LXC 102 | 🟢 Running | Bridge HomeKit |
| **Home Assistant** | QEMU 114 | ⏸️ Stopped | Domotique centrale |

---

## 📂 Documentation Détaillée

### Architecture
- **[Réseau](./architecture/network.md)** : Topologie réseau, VLANs, routage
- **[Stockage](./architecture/storage.md)** : ZFS, datasets, compression

### Hardware
- **[Compute](./hardware/compute.md)** : Serveur Proxmox (AMD Ryzen 7 5800U, 32 GB RAM)
- **[Storage](./hardware/storage.md)** : TrueNAS (Intel N100, 16 GB RAM, 2x 1TB WD Red)

### Sécurité
- **[Contrôle d'Accès](./security/access_control.md)** : Cloudflare Zero Trust, politiques d'accès

### Automation
- **[Backups](./automation/backups.md)** : Stratégies Proxmox + TrueNAS

---

## 🚀 Évolutions Futures

- [ ] **Réplication Off-Site** : Backups TrueNAS vers stockage distant
- [ ] **Haute Disponibilité** : Cluster Proxmox multi-nodes
- [ ] **Monitoring Avancé** : Alerting avec Prometheus + Alertmanager
- [ ] **CI/CD Pipeline** : GitLab CI ou Drone pour déploiements automatiques
- [ ] **Kubernetes** : Cluster K3s pour services stateless

---

## 🛠️ Technologies Utilisées

**Virtualisation & Conteneurs**
- Proxmox VE 9.0.6 (QEMU/KVM + LXC)
- Docker dans LXC dédié

**Stockage**
- TrueNAS Scale
- ZFS avec compression LZ4
- RAID 1 (mirroring)

**Réseau & Sécurité**
- Cloudflare Zero Trust (Tunnel + Access)
- Nginx Proxy Manager
- WireGuard VPN (à implémenter)

**Monitoring & Automation**
- Grafana + InfluxDB
- Uptime Kuma
- n8n Workflows

**Services Applicatifs**
- Arr Suite complète (Sonarr, Radarr, Prowlarr, Bazarr)
- Gitea (Git auto-hébergé)
- Homebridge (HomeKit)

---

## 📈 Métriques de Performance

**Consommation Ressources Proxmox** (moyenne)
- **CPU** : 1.5% utilisation idle
- **RAM** : 7 GB / 30.75 GB (23%)
- **Stockage** : 20 GB / 60 GB (33%)
- **Load Average** : 0.64, 0.70, 0.73

**Trafic Réseau** (depuis dernier boot)
- **Cloudflared** : 2.73 GB in / 2.39 GB out
- **qBittorrent** : 29.5 GB in / 1.55 GB out
- **Homebridge** : 18.5 GB in / 1.08 GB out

---

## 📝 Notes

Cette infrastructure évolue continuellement. La documentation est maintenue à jour via :
- **Métriques automatiques** : Récupérées via APIs Proxmox & TrueNAS
- **Infrastructure as Code** : Configuration versionnée sur Gitea
- **Monitoring continu** : Dashboards Grafana temps réel

---

**Dernière mise à jour** : 2025-11-16
**Généré automatiquement** : Données récupérées via MCP (Model Context Protocol) depuis APIs Proxmox & TrueNAS
