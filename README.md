# 🎥 MadaVisio L2 — Plateforme de Visioconférence Auto-Hébergée (Apache OpenMeetings)

![Status](https://img.shields.io/badge/Statut-Opérationnel-success)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![OpenMeetings](https://img.shields.io/badge/Apache_OpenMeetings-7.2.0-D22128)
![Kurento](https://img.shields.io/badge/Kurento_Media_Server-7.0.0-009688)
![OpenLDAP](https://img.shields.io/badge/Auth-OpenLDAP_1.5.0-blue)
![HTTPS](https://img.shields.io/badge/Security-TLS%2FHTTPS-green)

**Projet Académique L2 RSI (Réseaux et Systèmes Informatiques)** — Déploiement, sécurisation, intégration d'annuaire centralisé et personnalisation complète d'une solution de visioconférence collaborative auto-hébergée sous Linux.

---

## 📋 Table des matières

1. [Présentation du projet](#-présentation-du-projet)
2. [Architecture technique](#-architecture-technique)
3. [Fonctionnalités implémentées et validées](#-fonctionnalités-implémentées-et-validées)
4. [Arborescence du dépôt Git](#-arborescence-du-dépôt-git)
5. [Guide d'installation reproductible](#-guide-dinstallation-reproductible)
6. [Intégration OpenLDAP & Gestion des rôles](#-intégration-openldap--gestion-des-rôles)
7. [Personnalisation graphique (Branding)](#-personnalisation-graphique-branding)
8. [Captures d'écran & Démonstration](#-captures-décran--démonstration)
9. [Livrables académiques](#-livrables-académiques)

---

## 🎯 Présentation du projet

**MadaVisio L2** répond au besoin de souveraineté numérique et de maîtrise des flux multimédias d'une organisation ou d'un établissement d'enseignement. Basée sur **Apache OpenMeetings 7.2.0**, l'infrastructure est entièrement conteneurisée avec **Docker Compose** sur un hôte **Kali Linux**, garantissant :
* Une isolation stricte des services (application web, moteur média WebRTC, base de données relationnelle et annuaire LDAP).
* Une reproductibilité immédiate du déploiement.
* Des communications chiffrées de bout en bout via **HTTPS / TLS** et **DTLS-SRTP**.

---

## 🏗️ Architecture technique

L'infrastructure repose sur **4 conteneurs interconnectés** au sein d'un réseau bridge Docker dédié (`om-net`) :

```text
                        ┌──────────────────────────────────────────────┐
                        │           Navigateur Client (WebRTC)         │
                        │       https://localhost:5443/openmeetings    │
                        └──────────────────────┬───────────────────────┘
                                               │ HTTPS (TCP 5443) + WebRTC/SRTP
                                               ▼
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                                 HÔTE KALI LINUX (Docker)                             │
│                                                                                      │
│  ┌───────────────────────────────┐             ┌──────────────────────────────────┐  │
│  │       om-app (Tomcat)         │  WebSocket  │   om-kms (Kurento Media Server)  │  │
│  │  apache/openmeetings:min-7.2  │◄───────────►│  kurento/kurento-media-server:7  │  │
│  │  • Interface Wicket/Bootstrap │  (TCP 8888) │  • Flux Audio/Vidéo temps réel   │  │
│  │  • Certificat SSL (certs/)    │             │  • Partage d'écran & Enreg. MP4  │  │
│  └───────┬───────────────┬───────┘             └──────────────────────────────────┘  │
│          │               │                                                           │
│  JDBC    │               │ LDAP SIMPLEBIND (TCP 389)                                 │
│ (TCP     │               ▼                                                           │
│  3306)   │       ┌───────────────────────────────┐                                   │
│          │       │      om-ldap (OpenLDAP)       │                                   │
│          │       │     osixia/openldap:1.5.0     │                                   │
│          │       │  • Base : dc=entreprise,dc=mg │                                   │
│          │       │  • Comptes : rakoto, rasoa    │                                   │
│          │       └───────────────────────────────┘                                   │
│          ▼                                                                           │
│  ┌───────────────────────────────┐                                                   │
│  │      om-db (MariaDB 10.11)    │             Volumes persistants locaux :          │
│  │  • Base : openmeetings        │             • ./data/mariadb                      │
│  │  • Salles, utilisateurs,      │             • ./data/omdata (conf/om_ldap.cfg)    │
│  │    enregistrements, config    │             • ./certs (om.crt, om.key)            │
│  └───────────────────────────────┘                                                   │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

### Stack logicielle

| Composant | Image Docker / Version | Rôle dans l'architecture | Ports exposés |
| :--- | :--- | :--- | :--- |
| **Serveur Web & App** | `apache/openmeetings:min-7.2.0` | Portail Java/Tomcat, API, gestion des salles et tableau blanc | `5443` (HTTPS), `5080` (HTTP) |
| **Serveur Média** | `kurento/kurento-media-server:7.0.0` | Traitement WebRTC, routage flux audio/vidéo et enregistrement | `8888` (Interne Docker) |
| **Base de données** | `mariadb:10.11` | Stockage persistant des salles, droits, messages et métadonnées | `3306` (Interne Docker) |
| **Annuaire Centralisé** | `osixia/openldap:1.5.0` | Authentification centralisée des utilisateurs (`SIMPLEBIND`) | `389` (Interne Docker) |

---

## ✅ Fonctionnalités implémentées et validées

| Domaine | Fonctionnalité | Statut | Détails techniques |
| :--- | :--- | :---: | :--- |
| **Infrastructure** | Déploiement Linux + Java + MariaDB | ✅ | Stack Docker Compose orchestrée avec volumes persistants |
| **Infrastructure** | Kurento Media Server connecté | ✅ | Liaison WebSocket `ws://om-kms:8888/kurento` opérationnelle |
| **Sécurité** | Accès HTTPS fonctionnel | ✅ | Certificat TLS auto-signé (requis par WebRTC pour micro/caméra) |
| **Annuaire** | Authentification centralisée OpenLDAP | ✅ | Synchronisation via `om_ldap.cfg` (`dc=entreprise,dc=mg`) |
| **Administration** | Gestion fine des rôles & modération | ✅ | Rôles `Admin`, `Modérateur`, `Utilisateur`, `Invité`, actions `Mute`/`Kick` |
| **Personnalisation** | Branding complet (Logo, Dark Mode, FR) | ✅ | Thème sombre CSS, logo vectoriel/PNG, interface 100 % française |
| **Collaboration** | Planification & liens invités sécurisés | 🎬 | Calendrier intégré + génération d'URL protégée par mot de passe |
| **Visioconférence** | Audio/Vidéo multi-participants | 🎬 | Flux WebRTC bidirectionnels avec contrôle périphériques |
| **Collaboration** | Chat public & messagerie privée | 🎬 | Canal général de salle + onglets privés chiffrés |
| **Collaboration** | Partage d'écran & Tableau blanc | 🎬 | Import et annotation en direct de documents PDF/Office |
| **Archivage** | Enregistrement de session & Export MP4 | 🎬 | Capture serveur Kurento/FFmpeg, lecture web et téléchargement `.mp4` |

*(✅ Validé en configuration et tests unitaires | 🎬 Scénario de démonstration vidéo prêt)*

---

## 📂 Arborescence du dépôt Git

```text
projet-openmeetings/
├── .gitignore                                 # Exclusion des secrets (.env, *.key) et volumes de données
├── docker-compose.yml                         # Orchestration des 4 services (OpenMeetings, MariaDB, Kurento, OpenLDAP)
├── README.md                                  # Documentation technique complète et guide de déploiement
├── branding/                                  # Identité visuelle MadaVisio L2
│   ├── logo.svg                               # Logo vectoriel personnalisé
│   └── modern_theme.css                       # Feuille de style du thème sombre
├── certs/                                     # Certificats SSL/TLS (om.crt versionné, om.key ignoré)
│   └── om.crt
└── docs/                                      # Livrables documentaires du projet P01
    ├── captures_demonstration_madavisio.pdf   # Dossier complet des 27 captures annotées (28 pages)
    └── fiche_veille_openmeetings.pdf          # Fiche de veille technologique comparative
```

---

## 🚀 Guide d'installation reproductible

### Prérequis
* Système hôte Linux (Debian / Ubuntu / Kali Linux) avec **Docker** et **Docker Compose** installés.
* Minimum **4 Go de RAM** disponibles (8 Go recommandés pour la fluidité Kurento + navigateur).
* Ports libres : `5443` (HTTPS) et `5080` (HTTP).

### 1. Cloner le dépôt et préparer l'environnement
```bash
git clone https://github.com/<votre-utilisateur>/projet-openmeetings.git
cd projet-openmeetings
mkdir -p certs data/mariadb data/omdata/conf data/ldap
cp .env.example .env
```

### 2. Générer le certificat SSL/TLS auto-signé (obligatoire pour WebRTC)
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/om.key -out certs/om.crt \
  -subj "/C=MG/ST=Analamanga/L=Antananarivo/O=L2-RSI/CN=localhost"
```

### 3. Démarrer l'infrastructure Docker Compose
```bash
sudo docker compose up -d
sudo docker compose ps
```
> **Vérification :** Les 4 services (`om-db`, `om-kms`, `om-ldap`, `om-app`) doivent afficher l'état `Up`.

### 4. Finaliser l'assistant web et injecter la configuration LDAP
1. Accéder à `https://localhost:5443/openmeetings` et suivre l'assistant d'installation en renseignant les paramètres MariaDB définis dans `.env`.
2. Copier le connecteur LDAP dans le volume de configuration :
   ```bash
   cp ldap/om_ldap.cfg data/omdata/conf/om_ldap.cfg
   ```
3. Déclarer le domaine LDAP dans **Administration > Configuration LDAP** en pointant vers `om_ldap.cfg`.

---

## 🔐 Intégration OpenLDAP & Gestion des rôles

* **Base DN :** `dc=entreprise,dc=mg` (`ou=users,dc=entreprise,dc=mg`)
* **Méthode de liaison :** `SIMPLEBIND` via `cn=admin,dc=entreprise,dc=mg`
* **Matrice des permissions testée :**
  * **`admin` (Local)** & **`rakoto` (LDAP)** : Rôle **Administrateur** & **Modérateur** (accès complet à l'administration, création de salles, attribution des droits audio/vidéo, coupure micro `Mute` et expulsion `Kick`).
  * **`rasoa` (LDAP)** : Rôle **Utilisateur standard** (accès aux salles, chat, tableau blanc selon autorisation du modérateur, aucun accès au panneau d'administration).
  * **Participants externes** : Accès temporaire par **lien d'invitation protégé par mot de passe**.

---

## 🎨 Personnalisation graphique (Branding)

L'interface d'origine a été modernisée pour refléter l'identité de **MadaVisio L2** :
* **Identité :** `application.name` renommé en **MadaVisio L2**, interface basculée en **français** par défaut (`default.lang.id = 4`).
* **Logo personnalisé :** Redimensionnement et intégration dans `/opt/openmeetings/webapps/openmeetings/css/images/logo.png`.
* **Thème Sombre Intégral (`branding/modern_theme.css`) :** Surcharge CSS injectée dans `theme.min.css` appliquant une palette ardoise sombre (`#0b0f19` / `#111827`), des cartes aux coins arrondis et des accents cyan (`#0ea5e9`).

```bash
# Commande d'application du thème personnalisé dans le conteneur actif
sudo docker cp branding/logo_header.png om-app:/opt/openmeetings/webapps/openmeetings/css/images/logo.png
cat branding/modern_theme.css | sudo docker exec -i om-app tee -a /opt/openmeetings/webapps/openmeetings/css/theme.min.css > /dev/null
```

---

## 📸 Captures d'écran & Démonstration

L'ensemble des preuves visuelles chronologiques et commentées (déploiement Docker, accès HTTPS, annuaire OpenLDAP, personnalisation thème sombre MadaVisio L2 et gestion des accès) est regroupé dans le dossier PDF ci-dessous :

- 📄 **[Dossier complet des captures d'écran & démonstration commentées (PDF — 28 pages)](docs/captures_demonstration_madavisio.pdf)**
- 📘 **[Fiche de veille technologique — Apache OpenMeetings (PDF)](docs/fiche_veille_openmeetings.pdf)

## 📚 Livrables académiques

* **Fiche de veille technologique (PDF) :** [`docs/fiche_veille_openmeetings.pdf`](docs/fiche_veille_openmeetings.pdf) — Analyse comparative approfondie entre **Apache OpenMeetings**, **Jitsi Meet** et **BigBlueButton**, justification architecturale et préconisations de sécurité.
* **Dépôt Git public :** Code d'infrastructure (`docker-compose.yml`), scripts de configuration LDAP, thème CSS personnalisé et documentation complète.

---
*Réalisé dans le cadre du cursus Licence 2 — Réseaux et Systèmes Informatiques (L2 RSI).*