GUIDE D'INSTALLATION ET DE CONFIGURATION DE LA PLATEFORME CDNSS (DSPACE 9.x)
Projet : Centre de Documentation Numérique du Secteur Santé (CDNSS)
Version DSpace : 9.x (Backend)
Serveur Cible : Ubuntu/Debian (dspace.pheoc.cm)
Auteur : Djidjioua Hamadama Simon Pierre
Date : Novembre 2025

PHASE 1 : INSTALLATION DES PRÉREQUIS LOGICIELS ET DE DSPACE (SOCLE)
Ces commandes sont basées sur un serveur Ubuntu/Debian.
1. Mise à jour du système et outils de base
Connectez-vous en tant que votre utilisateur système (ex: pheocc).
[votre-user] - Mettez à jour le système :
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git wget curl vim software-properties-common unzip

2. Installation de Java 17 (Requis par DSpace 9)
DSpace 9 nécessite obligatoirement Java 17.
[votre-user] - Installation OpenJDK 17 :
sudo apt install -y openjdk-17-jdk

[votre-user] - Vérifiez l'installation :
java -version

Vous devriez voir : openjdk version "17.x.x"
[votre-user] - Configurez JAVA_HOME :
echo 'export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"' | sudo tee -a ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' | sudo tee -a ~/.bashrc
source ~/.bashrc

[votre-user] - Vérifiez JAVA_HOME :
echo $JAVA_HOME

Devrait afficher : /usr/lib/jvm/java-17-openjdk-amd64
3. Installation de Maven 3.9.11
Maven est utilisé pour compiler le code source Java de DSpace.
[votre-user] - Télécharger la dernière version :
cd /tmp
wget [https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz](https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz) 
sudo tar xzf apache-maven-3.9.11-bin.tar.gz -C /opt 
sudo ln -s /opt/apache-maven-3.9.11 /opt/maven

[votre-user] - Configurez Maven :
echo 'export M2_HOME=/opt/maven' | sudo tee -a ~/.bashrc
echo 'export PATH=$PATH:$M2_HOME/bin' | sudo tee -a ~/.bashrc
source ~/.bashrc

[votre-user] - Vérifiez Maven :
mvn -version

Devrait afficher : Apache Maven 3.9.11
4. Installation de Ant
Ant est utilisé pour le déploiement des fichiers binaires.
[votre-user] - Installez Ant :
sudo apt install -y ant

[votre-user] - Vérifiez Ant :
ant -version

5. Installation et Configuration de PostgreSQL
PostgreSQL est la base de données relationnelle de DSpace.
5.1 Installation
[votre-user] - Installez PostgreSQL :
sudo apt install -y postgresql postgresql-contrib

[votre-user] - Démarrez PostgreSQL :
sudo systemctl start postgresql
sudo systemctl enable postgresql

[votre-user] - Vérifiez le statut :
sudo systemctl status postgresql

Devrait afficher : active (running)
5.2 Création de la Base de Données et Extension pgcrypto
Cette étape est critique pour le support des UUIDs.
[votre-user] - Passez à l'utilisateur postgres :
sudo -i -u postgres

[postgres] - Vous êtes maintenant connecté en tant que postgres. Créez la base :
psql

[postgres via psql] - Dans le shell PostgreSQL, exécutez :
-- Créer l'utilisateur dspace avec un mot de passe fort
CREATE USER dspace WITH PASSWORD 'DSpace2024_CDNSS!Secure';

-- Créer la base de données
CREATE DATABASE dspace OWNER dspace ENCODING 'UTF8';

-- Donner tous les privilèges
GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;

-- Se connecter à la base dspace
\c dspace

-- Activer l'extension pgcrypto (requis par DSpace)
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Donner les droits sur le schéma public
GRANT ALL ON SCHEMA public TO dspace;

-- Quitter psql
\q

[postgres] - Retournez à votre utilisateur normal :
exit

[votre-user] - Vous êtes de retour en tant que votre utilisateur normal.
5.3 Configuration de l'Authentification (pg_hba.conf)
DSpace utilise une authentification par mot de passe (MD5), alors que PostgreSQL utilise souvent peer ou scram-sha-256 par défaut. Il faut harmoniser cela.
[votre-user] - Éditez le fichier pg_hba.conf :
sudo nano /etc/postgresql/16/main/pg_hba.conf

[votre-user] - Ajoutez ces lignes AVANT la ligne local all all peer :
# DSpace local connections
local   dspace          dspace                                  md5
host    dspace          dspace          127.0.0.1/32            md5
host    dspace          dspace          ::1/128                 md5

[votre-user] - Sauvegardez et quittez (ESC, puis :wq)
[votre-user] - Optimisez PostgreSQL pour DSpace :
sudo nano /etc/postgresql/16/main/postgresql.conf

[votre-user] - Trouvez et modifiez ces lignes (décommentez si nécessaire) :
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

[votre-user] - Redémarrez PostgreSQL :
sudo systemctl restart postgresql

[votre-user] - Testez la connexion :
psql -U dspace -h localhost -d dspace -W

Entrez le mot de passe : DSpace2024_CDNSS!Secure
Si connecté avec succès, tapez \q pour quitter
6. Installation d'Apache Solr 9.x
Solr est le moteur de recherche et d'indexation.
[votre-user] - Téléchargez Solr 9.10.0 (compatible DSpace 9.1) :
cd /tmp
wget [https://www.apache.org/dyn/closer.lua/solr/solr/9.10.0/solr-9.10.0.tgz?action=download](https://www.apache.org/dyn/closer.lua/solr/solr/9.10.0/solr-9.10.0.tgz?action=download)

[votre-user] - Move the tar to rename it :
mv solr-9.10.0.tgz\?action\=download solr-9.10.0.tgz

[votre-user] - Extrayez Solr :
sudo tar xzf solr-9.10.0.tgz -C /opt
sudo mv /opt/solr-9.10.0 /opt/solr

7. Création de l'Utilisateur Système DSpace
C'est l'utilisateur Linux qui exécutera l'application Tomcat.
[votre-user] - Créez l'utilisateur dspace :
sudo useradd -m -s /bin/bash dspace

[votre-user] - Définissez un mot de passe :
sudo passwd dspace
# Entrez un mot de passe sécurisé (ex: DSpaceUser2024!)

[votre-user] - Donnez les droits sudo à l'utilisateur dspace (optionnel, pour faciliter l'administration) :
sudo usermod -aG sudo dspace

[votre-user] - Donnez la propriété de Solr à l'utilisateur dspace :
sudo chown -R dspace:dspace /opt/solr

8. Téléchargement du Code Source DSpace
Nous passons à l'utilisateur dspace pour manipuler le code.
[votre-user] - Passez à l'utilisateur dspace :
sudo su - dspace

[dspace] - Vous êtes maintenant connecté en tant qu'utilisateur dspace.
sudo nano /home/dspace/.bashrc

[dspace] – Collez ceci à la fin du document :
export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
export M2_HOME="/opt/maven"
export PATH="$PATH:$JAVA_HOME/bin:$M2_HOME/bin"

[dspace] – Mettez à jour l'environnement :
source ~/.bashrc

[dspace] - Téléchargez DSpace :
cd ~
git clone https://github.com/DSpace/DSpace.git dspace-source
cd dspace-source

[dspace] - Basculez sur la version 9.1 :
git checkout dspace-9.1

[dspace] - Vérifiez que vous êtes sur la bonne version :
git branch

Devrait afficher : * dspace-9.1
PHASE 2 : CUSTOMISATION ET CONFIGURATION CDNSS
C'est ici que nous transformons DSpace générique en plateforme CDNSS, en appliquant les spécifications du MLD, des Workflows et de l'API.
9. Création des Dossiers de Configuration
Toujours en tant qu'utilisateur dspace :
mkdir -p /home/dspace/dspace-source/dspace/config/registries
mkdir -p /home/dspace/dspace-source/dspace/config/controlled-vocabularies
mkdir -p /home/dspace/dspace-source/dspace/config/entities

10. Création des Fichiers de Configuration Personnalisés
Créez les 7 fichiers suivants en copiant le contenu des fichiers joints fournis séparément.
10.1 local.cfg (Configuration Principale)
Ce fichier connecte la base de données, Solr, et configure l'API pour WordPress.
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/local.cfg

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint local.cfg.
10.2 cdnss-metadata.xml (Registre MLD)
Ce fichier définit les champs personnalisés (Status projet, Date AAR, etc.).
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/registries/cdnss-metadata.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint cdnss-metadata.xml.
10.3 relationship-types.xml (Définition des Relations - CRITIQUE)
Ce fichier est essentiel pour que DSpace comprenne les entités Projet et leurs liens.
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/entities/relationship-types.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint relationship-types.xml.
10.4 item-submission.xml (Entités & Processus)
Ce fichier définit les types d'objets (Projet, Publication) et lie les processus.
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/item-submission.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint item-submission.xml.
10.5 workflow.xml (Workflow de Validation)
Ce fichier définit les étapes (Cotation -> Traitement) et les rôles.
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/workflow.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint workflow.xml.
10.6 input-forms.xml (Formulaires)
Ce fichier définit les champs à afficher lors de la soumission.
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/input-forms.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint input-forms.xml.
10.7 types-etude.xml (Vocabulaire)
[dspace] - Créer le fichier :
nano /home/dspace/dspace-source/dspace/config/controlled-vocabularies/types-etude.xml

[dspace] - Contenu : Copier l'intégralité du contenu du fichier joint types-etude.xml.
11. Compilation et Installation
11.1 Compilation avec Maven
[dspace] - Compilez DSpace (cette étape peut prendre 10-20 minutes) :
cd /home/dspace/dspace-source
mvn clean package -Dmirrors.skip=true

ATTENDEZ que la compilation se termine. Vous devriez voir BUILD SUCCESS à la fin.
11.2 Installation (Ant)
Nous installons les fichiers compilés dans /dspace.
[dspace] - Créez le répertoire d'installation :
sudo mkdir -p /dspace
sudo chown dspace:dspace /dspace

[dspace] - Installez DSpace :
cd /home/dspace/dspace-source/dspace/target/dspace-installer
ant fresh_install

Attendez le message BUILD SUCCESSFUL.
[dspace] - Vérifiez que l'installation s'est bien passée :
ls -la /dspace

Vous devriez voir les dossiers: assetstore, bin, config, lib, solr, webapps, etc.
11.3 Chargement du Schéma de Métadonnées CDNSS
[dspace] - Chargez le schéma personnalisé :
/dspace/bin/dspace registry-loader -m /home/dspace/dspace-source/dspace/config/registries/cdnss-metadata.xml

PHASE 3 : DÉPLOIEMENT ET DÉMARRAGE DES SERVICES
12. Configuration et Démarrage de Solr
DSpace doit installer ses "cœurs" (index) dans Solr.
[votre-user] - En tant qu'utilisateur dspace :
# Copier les cœurs DSpace
cp -r /dspace/solr/* /opt/solr/server/solr/

# Démarrer Solr
/opt/solr/bin/solr start

Devrait voir: Started Solr server on port 8983
[dspace] - Vérifiez que Solr fonctionne :
curl http://localhost:8983/solr/admin/cores?action=STATUS

(if errors occur, you need to solve libraries errors: cp /opt/solr/modules/analysis-extras/lib/lucene-analysis-icu-9.12.3.jar /opt/solr/server/solr-webapp/webapp/WEB-INF/lib/ and cp /opt/solr/modules/analysis-extras/lib/icu4j-*.jar /opt/solr/server/solr-webapp/webapp/WEB-INF/lib/)
Vous devriez voir une réponse JSON avec les cores Solr.
NB: IF THE CORES ARE NOT CREATED AUTOMATICALLY, YOU CAN CREATE THEM MANUALLY: (Check CORES EXAMPLE FILES in the directory)
[dspace] - Créez les cores Solr pour DSpace 9.1 :
/opt/solr/bin/solr create -c authority -d /dspace/solr/authority
/opt/solr/bin/solr create -c oai -d /dspace/solr/oai
/opt/solr/bin/solr create -c search -d /dspace/solr/search
/opt/solr/bin/solr create -c statistics -d /dspace/solr/statistics

Pour chaque core, vous devriez voir: Created new core 'xxx'
[dspace] - Vérifiez que tous les cores sont créés :
curl -s "http://localhost:8983/solr/admin/cores?action=STATUS" | grep -o '"name":"[^"]*"'

Vous devriez voir: authority, oai, qaevent, search, statistics, suggestion.. etc
13. Installation de Tomcat 10
[dspace] - Retournez à votre utilisateur normal :
exit

[votre-user] - Téléchargez Tomcat 10 :
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.49/bin/apache-tomcat-10.1.49.tar.gz

[votre-user] - Installez Tomcat :
sudo tar xzf apache-tomcat-10.1.49.tar.gz -C /opt
sudo mv /opt/apache-tomcat-10.1.49 /opt/tomcat
sudo chown -R dspace:dspace /opt/tomcat

13.1 Configuration de la Mémoire Tomcat (setenv.sh)
[votre-user] - Configurez la mémoire Java pour Tomcat :
sudo nano /opt/tomcat/bin/setenv.sh

[votre-user] - Ajoutez ces lignes :
#!/bin/bash
# Configuration mémoire pour DSpace 9.1
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export CATALINA_OPTS="-Xmx2048m -Xms1024m -Dfile.encoding=UTF-8 -Djava.awt.headless=true"
export JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8"

[votre-user] - Rendez le fichier exécutable :
sudo chmod +x /opt/tomcat/bin/setenv.sh

13.2 Configuration du connecteur HTTP
[votre-user] - Éditez server.xml :
sudo nano /opt/tomcat/conf/server.xml

[votre-user] - Trouvez la section <Connector port="8080" et remplacez-la par :
<Connector port="8080" protocol="HTTP/1.1"
             connectionTimeout="20000"
             redirectPort="8443"
             maxThreads="200"
             minSpareThreads="10"
             maxHttpHeaderSize="65536"
             enableLookups="false"
             acceptCount="100"
             compression="on"
             compressionMinSize="2048"
             noCompressionUserAgents="gozilla, traviata" 
compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json,application/xml"
URIEncoding="UTF-8" />

13.3 Déploiement de l'Application (Lien Symbolique)
[votre-user] - create a symbolic link from Tomcat to your DSpace installation « server » folder :
sudo ln -s /dspace/webapps/server /opt/tomcat/webapps/server
sudo chown dspace:dspace /opt/tomcat/webapps/server

13.4 Création du service systemd pour Tomcat
[votre-user] - Créez le fichier service :
sudo nano /etc/systemd/system/tomcat.service

[votre-user] - Ajoutez ce contenu :
[Unit]
Description=Apache Tomcat pour DSpace 9.1
After=network.target postgresql.service

[Service]
Type=forking
User=dspace
Group=dspace

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_OPTS=-Xmx2048m -Xms1024m"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

[votre-user] - Sauvegardez et quittez
[votre-user] - Rechargez systemd :
sudo systemctl daemon-reload

[votre-user] - Activez le service Tomcat :
sudo systemctl enable tomcat

[votre-user] - Démarrez Tomcat :
sudo systemctl start tomcat

[votre-user] - Vérifiez le statut :
sudo systemctl status tomcat

Vous devriez voir active (running)
[votre-user] - Attendez 30-60 secondes que l'application se déploie, puis vérifiez les logs :
sudo tail -f /opt/tomcat/logs/catalina.out

Appuyez sur CTRL+C pour arrêter de suivre les logs Cherchez le message: Server startup in [xxxx] milliseconds
14. Initialisation de la Base de Données et des Données
C'est l'étape finale pour activer le MLD.
[votre-user] - Passez à l'utilisateur dspace :
sudo su - dspace

[dspace] - 1. Créer les tables de base :
/dspace/bin/dspace database migrate

[dspace] - 2. Créer l'administrateur :
/dspace/bin/dspace create-administrator

(Suivez les invites)
[dspace] - 3. CHARGER LE MLD (Métadonnées CDNSS) :
/dspace/bin/dspace registry-loader -metadata /dspace/config/registries/cdnss-metadata.xml

[dspace] - 4. Initialiser les Entités (POINT CRITIQUE) :
/dspace/bin/dspace initialize-entities -f /home/dspace/dspace-source/dspace/config/entities/relationship-types.xml

[dspace] - 5. Initialiser l'index de recherche :
/dspace/bin/dspace index-discovery -b

NB : (IF THERE ARE JAVA OR OTHER MODULES ERRORS DUE TO ENVIRONMENT, RUN THIS: export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin)

15. Démarrage Final
[votre-user] - Retournez à votre utilisateur normal :
exit

[votre-user] - Démarrez Tomcat :
sudo systemctl start tomcat

Votre backend est maintenant accessible à l'adresse : http://dspace.pheoc.cm:8080/server
Vous pouvez ensuite passer à la création de groupes :

16. Création des Groupes (Workflow)
Cette étape est cruciale pour que le Workflow CDNSS fonctionne.
Accédez à l'interface DSpace en tant qu'administrateur (via dspace.pheoc.cm:4000 ou l'URL de test).
Allez dans le menu Administration > Contrôle d'Accès > Groupes.
Créez deux nouveaux groupes (ou utilisez la ligne de commande ci-dessous pour être sûr) :
1.	Nom : Staff_DROS
2.	Nom : Point_Focal
[dspace] - Création via commande :
/dspace/bin/dspace dsprop -p "group.create=Staff_DROS"
/dspace/bin/dspace dsprop -p "group.create=Point_Focal"



PHASE DE TEST
ÉTAPE 1 : Vérification de l'API et des Entités
Nous devons vérifier que DSpace a bien "appris" votre nouveau langage (ProjetAAR, Institution, etc.). (Vous pouvez utiliser “curl” pour tester les entités)
Ouvrez votre navigateur et allez sur l'interface d'exploration de l'API (HAL Browser) : URL : http://dspace.pheoc.cm:8080/server Vous devriez voir une interface technique avec des liens.
Test des Entités : Cherchez le lien api/config/entitytypes et cliquez dessus (ou allez directement sur http://dspace.pheoc.cm:8080/server/api/core/entitytypes). Ce que vous devez voir : Dans la liste JSON qui s'affiche, vous devez trouver vos entités personnalisées : ProjetAAR, Institution, Auteur. Si vous ne les voyez pas, c'est que le fichier item-submission.xml n'a pas été pris en compte.
Test des Métadonnées : Allez sur http://dspace.pheoc.cm:8080/server/api/core/metadatafields/search/byFieldName?schema=cdnss&element=projet&qualifier=status 
Ce que vous devez voir : Une réponse JSON confirmant que le champ cdnss.project.status existe.
ÉTAPE 2 : Création de la Structure d'Accueil (Communautés & Collections)
DSpace est vide. Pour soumettre un projet, il faut créer l'arborescence.
Connectez-vous (si vous avez installé l'interface utilisateur Angular sur le port 4000) OU utilisez l'API/Ligne de commande si vous n'avez que le backend. Supposons que vous utilisez l'interface Admin (souvent installée par défaut pour la gestion).
Créez la structure suivante (qui servira votre Architecture) :
●	Communauté : "Recherche en Santé"
●	Sous-Communauté : "Division de la Recherche Opérationnelle (DROS)"
●	Collection : "Projets de Recherche AAR"
Action Critique : Lors de la création de cette collection, assignez-lui le "Type d'Entité" : ProjetAAR. C'est ce qui déclenchera votre formulaire personnalisé projet-form.
ÉTAPE 3 : Test de Soumission "À Blanc"
C'est le test ultime pour voir si vos formulaires (input-forms.xml) et votre vocabulaire (types-etude.xml) fonctionnent.
1.	Connectez-vous en tant qu'Admin.
2.	Lancez une nouvelle soumission dans la collection "Projets de Recherche AAR".
3.	Vérifiez le formulaire :
○	Est-ce que le champ "Type d’étude" apparaît ?
○	Est-ce que c'est une liste déroulante avec vos valeurs (Recherche fondamentale, clinique...) ?
○	Est-ce que le champ "Statut" est bien caché (comme configuré dans le XML) ?
4.	Remplissez et déposez le projet.
ÉTAPE 4 : Test du Workflow
1.	Une fois le projet soumis, il ne doit pas être publié tout de suite.
2.	Vérifiez qu'il est bien dans l'état "En attente de cotation".
3.	Si vous avez créé le groupe Staff_DROS (étape 16 du guide), ajoutez votre utilisateur à ce groupe.
4.	Vérifiez si vous voyez la tâche de validation dans votre tableau de bord ("My DSpace").

ÉTAPE 5 : Préparation pour le Développeur WordPress
Si tout ci-dessus fonctionne, votre Backend est prêt. Vous devez maintenant fournir les informations suivantes à l'équipe qui développe le Frontend WordPress :
1.	L'URL de l'API : http://dspace.pheoc.cm:8080/server
2.	L'UUID de la Collection "Projets" : (Récupérez l'ID de la collection "Projets de Recherche AAR" que vous avez créée). Le formulaire WordPress devra envoyer les données vers cet UUID.
3.	Le Mapping des Champs : Confirmez-leur que pour envoyer le statut, ils doivent utiliser la clé de métadonnée cdnss.project.status.

Création de la Structure et Test de Soumission
Maintenant que le backend est prêt, nous devons créer les conteneurs (Communautés/Collections) et tester si tout s'assemble (Formulaires + Entités + Workflow).
Voici la procédure à suivre immédiatement :
1. Créer la Structure (Via l'Interface Admin)
Connectez-vous à l'interface d'administration de DSpace (si vous avez installé l'UI, sinon via API/CLI).
1.	Créer une Communauté : "Recherche en Santé".
2.	Créer une Collection sous cette communauté : "Projets de Recherche AAR".
○	POINT CRITIQUE : Lors de la création de cette collection, vous devez lui assigner le Type d'Entité (Entity Type) : ProjetAAR.
○	Pourquoi ? C'est ce réglage qui dit à DSpace : "Quand quelqu'un soumet ici, utilise le formulaire projet-form défini dans item-submission.xml".
2. Test de Soumission (Validation Finale)
1.	En tant qu'administrateur, cliquez sur "Nouvelle Soumission" et choisissez la collection "Projets de Recherche AAR".
2.	Vérifiez le formulaire :
○	Voyez-vous les champs spécifiques comme "Type d'étude" ?
○	Est-ce que la liste déroulante "Type d'étude" contient bien vos valeurs (Fondamentale, Clinique...) ?
○	Est-ce que les champs cachés (Statut) sont bien invisibles ?
3.	Remplissez et soumettez.
4.	Vérifiez que le projet entre bien dans le Workflow (étape "En attente de cotation") et n'est pas publié immédiatement.
Si ce test passe, votre backend DSpace est officiellement prêt pour la production et pour être connecté à WordPress !




FICHIERS DE CONFIGURATION JOINT:
1. local.cfg (Configuration Principale)
2. cdnss-metadata.xml (Registre de Métadonnées)
3. relationship-types.xml (Définition des Relations)
4. item-submission.xml (Entités & Processus)
5. workflow.xml (Workflow de Validation)
6. input-forms.xml (Formulaires)
7. types-etude.xml (Vocabulaire)

