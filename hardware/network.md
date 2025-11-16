# Matériel : Réseau & Connexion Internet

## Vue d'Ensemble

Infrastructure réseau basée sur une connexion fibre optique FTTH 10 Gbit/s avec Freebox v7.

---

## 🌐 Connexion Internet

### Freebox v7 (r1) - Routeur Principal

| Caractéristique | Valeur |
|----------------|--------|
| **Modèle** | Freebox v7 révision 1 (fbxgw7r) |
| **Série** | 385502J230507583 |
| **MAC** | 20:66:CF:83:78:8E |
| **Firmware** | 4.9.12 (stable) |
| **Connexion** | FTTH 10 Gbit/s↓ / 900 Mbit/s↑ |
| **IP Publique** | 82.66.231.210 |
| **IPv6** | 2a01:e0a:2b6:2a10::1 /64 |
| **Uptime** | 11+ jours (très stable) |

### Capacités & Fonctionnalités

| Fonctionnalité | Statut | Détails |
|----------------|--------|---------|
| **WiFi 6E** | ✅ Actif | Tri-band 2.4/5/6 GHz |
| **Port SFP 10G** | ✅ Disponible | Fibre optique |
| **DECT** | ✅ Supporté | Téléphonie sans fil |
| **VM Support** | ✅ Oui | Virtualisation intégrée |
| **Home Automation** | ✅ Oui | Domotique Freebox |
| **Slots NAS** | 4 emplacements | Non utilisé |

### Performance Actuelle

**Trafic Total (depuis dernier reboot)** :
- **Download** : 990 GB
- **Upload** : 40.7 GB
- **Ratio** : 24:1 (typique usage média)

**Températures** :
- CPU Principal : 79°C 🟡
- CPU Secondaire : 79°C 🟡
- Ventilation : 1625-1723 RPM ✅

---

## 🔌 Infrastructure Réseau

### Topologie

```
Internet (Free FTTH)
    ↓ 10 Gbit/s
Freebox v7 (192.168.1.1)
    ↓ 1 Gbit/s Ethernet
Switch (non managé)
    ├── Proxmox VE (192.168.1.61)
    ├── TrueNAS (nas.lan)
    ├── NPM (192.168.1.186)
    └── Appareils clients
```

### Configuration DHCP

- **Serveur DHCP** : Freebox intégré
- **Plage** : 192.168.1.10 - 192.168.1.200
- **Sous-réseau** : 192.168.1.0/24
- **Passerelle** : 192.168.1.1
- **DNS** : Fournis par Free + Cloudflare 1.1.1.1

---

## 🛡️ Sécurité Réseau

### Politique Zero Trust

- **Aucun port ouvert** sur le routeur (pas de port forwarding)
- **Tous les services externes** passent par Cloudflare Tunnel
- **Firewall Freebox** : Actif
- **UPnP** : Désactivé (sécurité)
- **DMZ** : Non configuré

### Isolation

- Réseau LAN unique (192.168.1.0/24)
- Pas de VLANs actuellement (évolution future)
- Accès distant uniquement via VPN/Tunnel sécurisé

---

## 📊 Débits Mesurés

| Direction | Capacité | Utilisation Typique | Pic Observé |
|-----------|----------|---------------------|-------------|
| **Download** | 10 Gbit/s | 100-600 Mbit/s | ~800 Mbit/s |
| **Upload** | 900 Mbit/s | 5-50 Mbit/s | ~200 Mbit/s |

**Latence** : ~5-10ms (vers Paris)
**Jitter** : <2ms
**Perte de paquets** : 0%

---

## 🔧 Équipements Secondaires

| Rôle | Modèle | Spécifications |
|------|--------|----------------|
| **Switch** | Non managé | Gigabit (1000 Mbps) |
| **WiFi** | Intégré Freebox v7 | WiFi 6E tri-band |
| **Points d'accès** | Aucun | Couverture suffisante |

---

## 🚀 Évolutions Prévues

### Court Terme
- [ ] Switch managé 2.5G/10G pour segmentation
- [ ] VLANs (IoT, LAN, Lab)
- [ ] Monitoring réseau (ntopng)

### Moyen Terme
- [ ] Lien 10G direct Freebox → Proxmox
- [ ] Câblage Cat8 pour équipements critiques
- [ ] Backup WAN (4G/5G)

### Long Terme
- [ ] Upgrade vers Freebox Ultra
- [ ] Cluster réseau haute disponibilité
- [ ] SD-WAN pour multi-WAN

---

**Dernière mise à jour** : 16 novembre 2025
**Données récupérées via** : Freebox API
