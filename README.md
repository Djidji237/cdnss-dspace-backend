# Backend DSpace 9.1

[![DSpace Version](https://img.shields.io/badge/DSpace-9.1-blue.svg)](https://github.com/DSpace/DSpace)
[![Java Version](https://img.shields.io/badge/Java-17-orange.svg)](https://openjdk.org/projects/jdk/17/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📋 Description

Configuration et installation du backend DSpace 9.1 

Ce dépôt contient :
- Guide d'installation complet
- Fichiers de configuration personnalisés
- Schémas de métadonnées spécifiques au CDNSS
- Scripts utilitaires de gestion
- Documentation API

## 🎯 Fonctionnalités Principales

- ✅ Gestion des projets de recherche (AAR - Autorisation Administrative de Recherche)
- ✅ Workflow de validation multi-étapes
- ✅ Registre des essais cliniques
- ✅ Métadonnées personnalisées pour le secteur santé
- ✅ Vocabulaires contrôlés (thématiques, types d'études, régions)
- ✅ API REST pour intégration WordPress
- ✅ Support One Health
- ✅ Cartographie des projets de recherche

## 🔧 Technologies

- **DSpace** : 9.1
- **Java** : OpenJDK 17
- **PostgreSQL** : 14+
- **Apache Solr** : 9.10.0
- **Apache Tomcat** : 10.1.49
- **Maven** : 3.9.11

## 📚 Documentation

- [Guide d'Installation Complet](docs/GUIDE_INSTALLATION.md)
- [Configuration](docs/CONFIGURATION.md)
- [Documentation API](docs/API.md)
- [FAQ & Dépannage](docs/FAQ.md)

## 🚀 Installation Rapide

### Prérequis

- Ubuntu 20.04+ ou Debian 11+
- 4 GB RAM minimum (8 GB recommandé)
- 50 GB d'espace disque
- Accès root ou sudo

### Étapes Principales
```bash
# 1. Cloner le dépôt
git clone https://github.com/VOTRE-USERNAME/cdnss-dspace-backend.git
cd cdnss-dspace-backend

# 2. Suivre le guide d'installation
# Voir docs/GUIDE_INSTALLATION.md pour les instructions détaillées

# 3. Copier et configurer local.cfg
cp config/local.cfg.example /dspace/config/local.cfg
# Éditez et adaptez selon votre environnement

# 4. Démarrer les services
sudo systemctl start tomcat
```

## 📁 Structure du Projet
```
cdnss-dspace-backend/
├── config/                          # Fichiers de configuration
│   ├── registries/                  # Schémas de métadonnées
│   │   └── cdnss-metadata.xml
│   ├── entities/                    # Définition des entités
│   │   └── relationship-types.xml
│   ├── controlled-vocabularies/     # Vocabulaires contrôlés
│   │   └── types-etude.xml
│   ├── input-forms/                 # Formulaires de saisie
│   │   └── input-forms.xml
│   ├── workflow/                    # Configuration du workflow
│   │   └── workflow.xml
│   ├── item-submission.xml          # Configuration de soumission
│   └── local.cfg.example            # Configuration principale (exemple)
├── scripts/                         # Scripts utilitaires
│   ├── cdnss-start                  # Démarrage des services
│   ├── cdnss-stop                   # Arrêt des services
│   ├── cdnss-status                 # Vérification du statut
│   ├── cdnss-backup                 # Sauvegarde
│   └── cdnss-get-token              # Authentification API
├── docs/                            # Documentation
│   ├── GUIDE_INSTALLATION.md
│   ├── CONFIGURATION.md
│   ├── API.md
│   └── FAQ.md
├── examples/                        # Exemples
│   └── api-examples.md
└── README.md                        # Ce fichier
```

## 🔗 Endpoints API Principaux

| Endpoint | Description |
|----------|-------------|
| `/server/api` | Root de l'API REST |
| `/server/api/core/communities` | Communautés |
| `/server/api/core/collections` | Collections |
| `/server/api/core/items` | Items (documents, projets) |
| `/server/api/discover/search/objects` | Recherche |
| `/server/api/authn/login` | Authentification |

## 👥 Rôles et Permissions

- **Administrateur** : Accès complet au système
- **Staff DROS** : Validation des projets AAR, gestion documentaire
- **Point Focal** : Relecture institutionnelle
- **Chercheur** : Soumission de projets AAR
- **Visiteur** : Consultation publique

## 🔐 Sécurité

- ✅ Authentification JWT
- ✅ CSRF Protection activée
- ✅ HTTPS recommandé en production
- ✅ Sauvegardes automatiques chiffrées
- ✅ Gestion granulaire des permissions

## 📊 Schéma de Métadonnées CDNSS

Métadonnées personnalisées pour les projets de recherche :

- `cdnss.project.code` : Code unique du projet AAR
- `cdnss.project.status` : Statut (soumis, validé, rejeté, achevé)
- `cdnss.project.ethicalClearanceNumber` : Numéro de clairance éthique
- `cdnss.coverage.region` : Région(s) de mise en œuvre
- `cdnss.onehealth.component` : Composantes One Health
- Et bien d'autres...

[Voir la liste complète](docs/METADATA_SCHEMA.md)

## 🤝 Contribution

Les contributions sont les bienvenues ! Veuillez :

1. Forker le projet
2. Créer une branche (`git checkout -b feature/amelioration`)
3. Commiter vos changements (`git commit -m 'Ajout de fonctionnalité'`)
4. Pusher vers la branche (`git push origin feature/amelioration`)
5. Ouvrir une Pull Request

## 📝 License

Ce projet est sous licence MIT - voir le fichier [LICENSE](LICENSE) pour plus de détails.

## 👨‍💻 Auteurs

- Djidjioua Hamadama Simon Pierre

## 📧 Contact

Pour toute question ou support :
- Email : simoniopierre@gmail.com
- Issues GitHub : [Créer une issue](https://github.com/VOTRE-USERNAME/cdnss-dspace-backend/issues)

## 🙏 Remerciements

- DSpace Community
- PHEOC
- Tous les contributeurs

---

**Note** : Ce projet est en développement actif. Consultez la [documentation](docs/) pour les dernières mises à jour.
