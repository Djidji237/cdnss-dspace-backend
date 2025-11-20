# GUIDE D'INSTALLATION ET DE CONFIGURATION DE LA PLATEFORME \[NOM_PROJET\] (DSPACE 9.x)

Projet : \[VOTRE PROJET / CENTRE DE DOCUMENTATION\]

Version DSpace : 9.x (Backend)

Serveur Cible : Ubuntu/Debian (dspace.\[votre-domaine\].com)

Auteur : Djidjioua Hamadama Simon Pierre

Date : Novembre 2025

## PHASE 1 : INSTALLATION DES PRÉREQUIS LOGICIELS ET DE DSPACE (SOCLE)

_Ces commandes sont basées sur un serveur Ubuntu/Debian._

### 1\. Mise à jour du système et outils de base

Connectez-vous en tant que votre utilisateur système (ex: \[votre-user\]).

**\[votre-user\]** - Mettez à jour le système :

sudo apt update && sudo apt upgrade -y  
sudo apt install -y build-essential git wget curl vim software-properties-common unzip  

### 2\. Installation de Java 17 (Requis par DSpace 9)

DSpace 9 nécessite obligatoirement Java 17.

**\[votre-user\]** - Installation OpenJDK 17 :

sudo apt install -y openjdk-17-jdk  

**\[votre-user\]** - Vérifiez l'installation :

java -version  

_Vous devriez voir : openjdk version "17.x.x"_

**\[votre-user\]** - Configurez JAVA_HOME :

echo 'export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"' | sudo tee -a ~/.bashrc  
echo 'export PATH=$PATH:$JAVA_HOME/bin' | sudo tee -a ~/.bashrc  
source ~/.bashrc  

**\[votre-user\]** - Vérifiez JAVA_HOME :

echo $JAVA_HOME  

_Devrait afficher : /usr/lib/jvm/java-17-openjdk-amd64_

### 3\. Installation de Maven 3.9.11

Maven est utilisé pour compiler le code source Java de DSpace.

**\[votre-user\]** - Télécharger la dernière version :

cd /tmp  
wget https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz  
sudo tar xzf apache-maven-3.9.11-bin.tar.gz -C /opt  
sudo ln -s /opt/apache-maven-3.9.11 /opt/maven  

**\[votre-user\]** - Configurez Maven :

echo 'export M2_HOME=/opt/maven' | sudo tee -a ~/.bashrc  
echo 'export PATH=$PATH:$M2_HOME/bin' | sudo tee -a ~/.bashrc  
source ~/.bashrc  

**\[votre-user\]** - Vérifiez Maven :

mvn -version  

_Devrait afficher : Apache Maven 3.9.11_

### 4\. Installation de Ant

Ant est utilisé pour le déploiement des fichiers binaires.

**\[votre-user\]** - Installez Ant :

sudo apt install -y ant  

**\[votre-user\]** - Vérifiez Ant :

ant -version  

### 5\. Installation et Configuration de PostgreSQL

PostgreSQL est la base de données relationnelle de DSpace.

#### 5.1 Installation

**\[votre-user\]** - Installez PostgreSQL :

sudo apt install -y postgresql postgresql-contrib  

**\[votre-user\]** - Démarrez PostgreSQL :

sudo systemctl start postgresql  
sudo systemctl enable postgresql  

**\[votre-user\]** - Vérifiez le statut :

sudo systemctl status postgresql  

_Devrait afficher : active (running)_

#### 5.2 Création de la Base de Données et Extension pgcrypto

_Cette étape est critique pour le support des UUIDs._

**\[votre-user\]** - Passez à l'utilisateur postgres :

sudo -i -u postgres  

**\[postgres\]** - Vous êtes maintenant connecté en tant que postgres. Créez la base :

psql  

**\[postgres via psql\]** - Dans le shell PostgreSQL, exécutez :

\-- Créer l'utilisateur dspace avec un mot de passe fort  
CREATE USER dspace WITH PASSWORD '\[VOTRE_MOT_DE_PASSE_SECURISE\]';  
<br/>\-- Créer la base de données  
CREATE DATABASE dspace OWNER dspace ENCODING 'UTF8';  
<br/>\-- Donner tous les privilèges  
GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;  
<br/>\-- Se connecter à la base dspace  
\\c dspace  
<br/>\-- Activer l'extension pgcrypto (requis par DSpace)  
CREATE EXTENSION IF NOT EXISTS pgcrypto;  
<br/>\-- Donner les droits sur le schéma public  
GRANT ALL ON SCHEMA public TO dspace;  
<br/>\-- Quitter psql  
\\q  

**\[postgres\]** - Retournez à votre utilisateur normal :

exit  

#### 5.3 Configuration de l'Authentification (pg_hba.conf)

DSpace utilise une authentification par mot de passe (MD5). Il faut harmoniser cela.

**\[votre-user\]** - Éditez le fichier pg_hba.conf (vérifiez le chemin de version, ex: 14 ou 16) :

sudo nano /etc/postgresql/16/main/pg_hba.conf  

**\[votre-user\]** - Ajoutez ces lignes AVANT la ligne local all all peer :

\# DSpace local connections  
local dspace dspace md5  
host dspace dspace 127.0.0.1/32 md5  
host dspace dspace ::1/128 md5  

**\[votre-user\]** - Sauvegardez et quittez.

**\[votre-user\]** - Optimisez PostgreSQL pour DSpace (postgresql.conf) :

sudo nano /etc/postgresql/16/main/postgresql.conf  

Modifiez les valeurs suivantes (décommentez si nécessaire) :

max_connections = 200  
shared_buffers = 512MB  
effective_cache_size = 2GB  
maintenance_work_mem = 128MB  
checkpoint_completion_target = 0.9  
wal_buffers = 16MB  
default_statistics_target = 100  
random_page_cost = 1.1  
effective_io_concurrency = 200  
work_mem = 4MB  

**\[votre-user\]** - Redémarrez PostgreSQL :

sudo systemctl restart postgresql  

**\[votre-user\]** - Testez la connexion :

psql -U dspace -h localhost -d dspace -W  

_Entrez le mot de passe : \[VOTRE_MOT_DE_PASSE_SECURISE\]. Tapez \\q pour quitter._

### 6\. Installation d'Apache Solr 9.x

**\[votre-user\]** - Téléchargez Solr 9.10.0 :

cd /tmp  
wget https://www.apache.org/dyn/closer.lua/solr/solr/9.10.0/solr-9.10.0.tgz?action=download  
mv solr-9.10.0.tgz\\?action\\=download solr-9.10.0.tgz  

**\[votre-user\]** - Extrayez Solr :

sudo tar xzf solr-9.10.0.tgz -C /opt  
sudo mv /opt/solr-9.10.0 /opt/solr  

### 7\. Création de l'Utilisateur Système DSpace

**\[votre-user\]** - Créez l'utilisateur dspace :

sudo useradd -m -s /bin/bash dspace  
sudo passwd dspace  
\# Entrez un mot de passe sécurisé  
sudo usermod -aG sudo dspace  
sudo chown -R dspace:dspace /opt/solr  

### 8\. Téléchargement du Code Source DSpace

**\[votre-user\]** - Passez à l'utilisateur dspace :

sudo su - dspace  

**\[dspace\]** - Configurez l'environnement :

nano /home/dspace/.bashrc  

Ajoutez à la fin :

export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"  
export M2_HOME="/opt/maven"  
export PATH="$PATH:$JAVA_HOME/bin:$M2_HOME/bin"  

Activez :

source ~/.bashrc  

**\[dspace\]** - Téléchargez DSpace :

cd ~  
git clone \[https://github.com/DSpace/DSpace.git\](https://github.com/DSpace/DSpace.git) dspace-source  
cd dspace-source  
git checkout dspace-9.1  
git branch  

_Devrait afficher : \* dspace-9.1_

## PHASE 2 : CUSTOMISATION ET CONFIGURATION \[NOM_PROJET\]

C'est ici que nous transformons DSpace générique en votre plateforme personnalisée.

### 9\. Création des Dossiers de Configuration

**\[dspace\]** :

mkdir -p /home/dspace/dspace-source/dspace/config/registries  
mkdir -p /home/dspace/dspace-source/dspace/config/controlled-vocabularies  
mkdir -p /home/dspace/dspace-source/dspace/config/entities  

### 10\. Création des Fichiers de Configuration Personnalisés

Créez les 7 fichiers suivants (les contenus doivent être préparés à l'avance).

#### 10.1 local.cfg (Configuration Principale)

nano /home/dspace/dspace-source/dspace/config/local.cfg  

_Copier l'intégralité du contenu de votre fichier local.cfg._

#### 10.2 custom-metadata.xml (Registre de Métadonnées)

_Ce fichier définit les champs personnalisés (ex: Status projet, Date, etc.)._

nano /home/dspace/dspace-source/dspace/config/registries/custom-metadata.xml  

_Copier le contenu._

#### 10.3 relationship-types.xml (Définition des Relations - CRITIQUE)

nano /home/dspace/dspace-source/dspace/config/entities/relationship-types.xml  

_Copier le contenu._

#### 10.4 item-submission.xml (Entités & Processus)

nano /home/dspace/dspace-source/dspace/config/item-submission.xml  

_Copier le contenu._

#### 10.5 workflow.xml (Workflow de Validation)

nano /home/dspace/dspace-source/dspace/config/workflow.xml  

_Copier le contenu._

#### 10.6 input-forms.xml (Formulaires)

nano /home/dspace/dspace-source/dspace/config/input-forms.xml  

_Copier le contenu._

#### 10.7 types-etude.xml (Vocabulaire)

nano /home/dspace/dspace-source/dspace/config/controlled-vocabularies/types-etude.xml  

_Copier le contenu._

### 11\. Compilation et Installation

#### 11.1 Compilation avec Maven

**\[dspace\]** - Compilez :

cd /home/dspace/dspace-source  
mvn clean package -Dmirrors.skip=true  

_ATTENDEZ le message BUILD SUCCESS._

#### 11.2 Installation (Ant)

**\[dspace\]** - Installez :

sudo mkdir -p /dspace  
sudo chown dspace:dspace /dspace  
cd /home/dspace/dspace-source/dspace/target/dspace-installer  
ant fresh_install  

_ATTENDEZ le message BUILD SUCCESSFUL._

#### 11.3 Chargement du Schéma de Métadonnées

**\[dspace\]** :

/dspace/bin/dspace registry-loader -m /home/dspace/dspace-source/dspace/config/registries/custom-metadata.xml  

## PHASE 3 : DÉPLOIEMENT ET DÉMARRAGE DES SERVICES

### 12\. Configuration et Démarrage de Solr

**\[dspace\]** :

cp -r /dspace/solr/\* /opt/solr/server/solr/  
/opt/solr/bin/solr start  

_Vérifiez : curl http://localhost:8983/solr/admin/cores?action=STATUS_

Si les cœurs ne sont pas créés automatiquement :

/opt/solr/bin/solr create -c authority -d /dspace/solr/authority  
/opt/solr/bin/solr create -c oai -d /dspace/solr/oai  
/opt/solr/bin/solr create -c search -d /dspace/solr/search  
/opt/solr/bin/solr create -c statistics -d /dspace/solr/statistics  

### 13\. Installation de Tomcat 10

**\[dspace\]** -> exit (retour à **\[votre-user\]**)

**\[votre-user\]** - Installez Tomcat :

cd /tmp  
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.49/bin/apache-tomcat-10.1.49.tar.gz  
sudo tar xzf apache-tomcat-10.1.49.tar.gz -C /opt  
sudo mv /opt/apache-tomcat-10.1.49 /opt/tomcat  
sudo chown -R dspace:dspace /opt/tomcat  

#### 13.1 Configuration (setenv.sh)

sudo nano /opt/tomcat/bin/setenv.sh  

Ajoutez :

#!/bin/bash  
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  
export CATALINA_OPTS="-Xmx2048m -Xms1024m -Dfile.encoding=UTF-8 -Djava.awt.headless=true"  
export JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8"  

Rendez exécutable : sudo chmod +x /opt/tomcat/bin/setenv.sh

#### 13.2 Configuration du connecteur HTTP (server.xml)

Éditez /opt/tomcat/conf/server.xml et mettez à jour le connecteur port 8080 avec URIEncoding="UTF-8".

#### 13.3 Déploiement

sudo ln -s /dspace/webapps/server /opt/tomcat/webapps/server  
sudo chown dspace:dspace /opt/tomcat/webapps/server  

#### 13.4 Service Systemd

Créez /etc/systemd/system/tomcat.service (contenu standard Tomcat pour DSpace).

Activez et démarrez :

sudo systemctl daemon-reload  
sudo systemctl enable tomcat  
sudo systemctl start tomcat  

### 14\. Initialisation de la Base de Données et des Données

**\[dspace\]** :

/dspace/bin/dspace database migrate  
/dspace/bin/dspace create-administrator  

\# Chargez le schéma  
/dspace/bin/dspace registry-loader -metadata /dspace/config/registries/custom-metadata.xml  

\# Initialiser les Entités (CRITIQUE)  
/dspace/bin/dspace initialize-entities -f /home/dspace/dspace-source/dspace/config/entities/relationship-types.xml  

\# Indexer  
/dspace/bin/dspace index-discovery -b  

### 15\. Démarrage Final

Votre backend est accessible à : http://\[votre-domaine\]:8080/server

### 16\. Création des Groupes (Workflow)

Créez les groupes nécessaires au workflow :

/dspace/bin/dspace dsprop -p "group.create=Staff_Reviewer"  
/dspace/bin/dspace dsprop -p "group.create=Point_Focal"  

## PHASE DE TEST

**ÉTAPE 1 : Vérification de l'API**

Testez http://\[votre-domaine\]:8080/server/api/core/entitytypes. Vous devez voir vos entités personnalisées (ex: Projet, Institution).

**ÉTAPE 2 : Création de la Structure**

1.  Créez une Communauté : "Recherche"
2.  Créez une Collection : "Projets de Recherche"
3.  **CRITIQUE :** Assignez le "Type d'Entité" : Projet à cette collection.

ÉTAPE 3 : Test de Soumission

En tant qu'Admin, lancez une soumission. Vérifiez que vos formulaires personnalisés apparaissent.

ÉTAPE 4 : Test du Workflow

Soumettez un projet. Vérifiez qu'il tombe en "Attente de cotation" et n'est pas publié immédiatement.

**ÉTAPE 5 : Préparation Frontend**

Fournissez à l'équipe Web/WordPress :

1.  L'URL API : http://\[votre-domaine\]:8080/server
2.  L'UUID de la Collection "Projets".
