# 🛡️ Cisco Zone-Based Policy Firewall (ZPF) Configuration

## 📋 Vue d'ensemble

Implémentation complète d'un pare-feu ZPF (Zone-Based Policy Firewall) sur un routeur Cisco pour sécuriser un réseau d'entreprise. Ce projet démontre la configuration avancée de zones de sécurité, de politiques de filtrage, et la validation des règles de pare-feu entre réseaux internes et externes.

**Contexte :** Travaux pratiques académiques - Master Sécurité IT & Big Data

---

## 🎯 Objectifs du projet

✅ **Segmentation réseau** : Création de zones de sécurité (IN-ZONE / OUT-ZONE)  
✅ **Contrôle d'accès** : Autorisation du trafic sortant, blocage du trafic entrant non sollicité  
✅ **Inspection stateful** : Configuration de l'inspection de paquets pour les sessions établies  
✅ **Validation** : Tests de connectivité (ping, SSH, HTTP) entre zones  
✅ **Sécurisation SSH** : Désactivation de Telnet et configuration RSA 2048-bit

---

## 🏗️ Architecture réseau

```
[PC-C]───────[R3 Firewall]───────[R2]───────[PC-A Server]
192.168.3.3   G0/1 | S0/0/1    10.2.2.2      192.168.1.3
              ▼         ▼
           IN-ZONE   OUT-ZONE
```

### Zones de sécurité définies
| Zone | Interface | Réseau | Fonction |
|------|-----------|--------|----------|
| **IN-ZONE** | GigabitEthernet0/1 | 192.168.3.0/24 | Réseau interne (trusted) |
| **OUT-ZONE** | Serial0/0/1 | 10.2.2.0/24 | Réseau externe (untrusted) |

---

## 🔧 Technologies et concepts

- **Routeur Cisco** : Configuration IOS avec package SecurityK9
- **ZPF (Zone-Based Policy Firewall)** : Pare-feu de nouvelle génération Cisco
- **ACL étendues** : Identification du trafic réseau
- **Class-map & Policy-map** : Définition des règles de sécurité
- **Zone-pair** : Association de politiques entre zones source/destination
- **SSH v2** : Chiffrement RSA 2048-bit pour l'administration à distance
- **Inspection stateful** : Suivi des connexions établies

---

## 📄 Documentation

📥 **[Télécharger le rapport complet (PDF)](docs/ZPF-Config.pdf)**

---

## 🚀 Étapes de configuration

### 1️⃣ **Sécurisation des accès routeur**
```cisco
Router(config)# hostname R3
R3(config)# enable secret ciscoenpa55
R3(config)# line console 0
R3(config-line)# password ciscoconpa55
R3(config-line)# login
R3(config)# line vty 0 15
R3(config-line)# password ciscovtypa55
R3(config-line)# login local
R3(config-line)# transport input ssh
```

### 2️⃣ **Activation du package de sécurité**
```cisco
R3# show version                  # Vérification de securityk9
R3(config)# license boot module c1900 technology-package securityk9
R3# copy running-config startup-config
R3# reload
```

### 3️⃣ **Création des zones de sécurité**
```cisco
R3(config)# zone security IN-ZONE
R3(config)# zone security OUT-ZONE
```

### 4️⃣ **Définition du trafic autorisé (ACL + Class-map)**
```cisco
R3(config)# access-list 101 permit ip 192.168.3.0 0.0.0.255 any
R3(config)# class-map type inspect match-all IN-NET-CLASS-MAP
R3(config-cmap)# match access-group 101
```

### 5️⃣ **Configuration de la politique de sécurité**
```cisco
R3(config)# policy-map type inspect IN-2-OUT-PMAP
R3(config-pmap)# class type inspect IN-NET-CLASS-MAP
R3(config-pmap-c)# inspect
```

### 6️⃣ **Application des politiques (Zone-pair)**
```cisco
R3(config)# zone-pair security IN-2-OUT-ZPAIR source IN-ZONE destination OUT-ZONE
R3(config-sec-zone-pair)# service-policy type inspect IN-2-OUT-PMAP
```

### 7️⃣ **Association des interfaces aux zones**
```cisco
R3(config)# interface g0/1
R3(config-if)# zone-member security IN-ZONE

R3(config)# interface s0/0/1
R3(config-if)# zone-member security OUT-ZONE
```

---

## ✅ Résultats des tests

### ✔️ **Trafic sortant (IN-ZONE → OUT-ZONE) : AUTORISÉ**

| Test | Source (PC-C) | Destination | Résultat |
|------|---------------|-------------|----------|
| **ICMP** | 192.168.3.3 | PC-A (192.168.1.3) | ✅ Succès |
| **SSH** | 192.168.3.3 | R2 (10.2.2.2) | ✅ Connexion établie |
| **HTTP** | 192.168.3.3 | PC-A (192.168.1.3) | ✅ Page web accessible |

### ❌ **Trafic entrant (OUT-ZONE → IN-ZONE) : BLOQUÉ**

| Test | Source | Destination (PC-C) | Résultat |
|------|--------|-------------------|----------|
| **ICMP** | PC-A (192.168.1.3) | 192.168.3.3 | ❌ Échec (timeout) |
| **ICMP** | R2 (10.2.2.2) | 192.168.3.3 | ❌ Échec (timeout) |

---

## 🔍 Commandes de vérification

### Afficher les sessions actives
```cisco
R3# show policy-map type inspect zone-pair sessions
```
**Exemple de sortie :**
```
Established Sessions
  Session 648201E8 (192.168.3.3:1025)=>(10.2.2.2:22) ssh :
    Created 00:00:12, Last heard 00:00:08
    Bytes sent (initiator:responder) [480:1456]
```

### Vérifier les zones configurées
```cisco
R3# show zone security
R3# show zone-pair security
```

### Statistiques de la politique
```cisco
R3# show policy-map type inspect zone-pair IN-2-OUT-ZPAIR
```

---

## 📊 Concepts clés démontrés

### 🔐 **Principe de moindre privilège**
Par défaut, tout trafic entre zones est **bloqué** sauf autorisation explicite.

### 🛠️ **Inspection stateful**
Le pare-feu maintient l'état des connexions établies :
- Trafic sortant initié → sessions retour autorisées automatiquement
- Trafic entrant non sollicité → bloqué

### 📝 **Structure hiérarchique ZPF**
```
Zone ──> Class-map ──> Policy-map ──> Zone-pair ──> Interface
```

---

## 🔒 Sécurité SSH implémentée

### Configuration complète
```cisco
R2(config)# hostname R2
R2(config)# ip domain-name lsi.com
R2(config)# crypto key generate rsa
How many bits in the modulus [512]: 2048
R2(config)# ip ssh version 2
R2(config)# username admin password Adminpa55
R2(config)# line vty 0 15
R2(config-line)# login local
R2(config-line)# transport input ssh
```

### Test depuis client
```bash
C:\> ssh -l admin 10.2.2.2
Password: Adminpa55
```

---

## 🎓 Compétences techniques développées

- ✅ Configuration avancée de routeurs Cisco IOS
- ✅ Design et implémentation de politiques de sécurité réseau
- ✅ Maîtrise des pare-feux stateful (ZPF)
- ✅ Gestion des ACL et classification de trafic
- ✅ Sécurisation des accès administratifs (SSH v2, RSA)
- ✅ Troubleshooting et validation de configurations réseau
- ✅ Documentation technique professionnelle

---

## 📚 Améliorations possibles

- [ ] Configuration de zones DMZ pour serveurs publics
- [ ] Implémentation de politiques granulaires par protocole
- [ ] Logging centralisé des événements de sécurité
- [ ] Configuration de rate-limiting pour prévenir le DoS
- [ ] Intégration avec des systèmes IPS/IDS
- [ ] Déploiement de VPN site-to-site avec ZPF

---

## 👤 Auteur

**Chaimae Bouassab**  
Master Sécurité IT & Big Data  
Université Abdelmalek Essaadi - FST Tanger

📧 [Email](mailto:ton-email)  
💼 [LinkedIn](ton-profil-linkedin)  
🌐 [Portfolio](ton-site-web)

---

## 📖 Références

- [Cisco ZPF Configuration Guide](https://www.cisco.com/c/en/en/support/docs/security/ios-firewall/98628-zone-design-guide.html)
- Cours : Cryptographie & Sécurité Services - Prof. A. GHADI
- Documentation Cisco IOS Security

---

## 📝 License

Projet académique - 2024/2025  
Documentation disponible sous [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

---

⭐ **Si ce projet vous est utile, n'hésitez pas à le star !**