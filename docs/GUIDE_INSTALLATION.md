**PHASE 1 : INSTALLATION DES PREREQUIS LOGICIELS ET DE DSPACE**

Ces commandes sont basées sur un serveur **Ubuntu/Debian**.

- **Mettre à jour votre système :**

**\[votre-user\]** - Mettez à jour le système :

sudo apt update && sudo apt upgrade -y

sudo apt install -y build-essential git wget curl vim software-properties-common unzip

- Installer Java 17 (Requis par DSpace 9) :

sudo apt install -y openjdk-17-jdk

\# Vérifiez l'installation

java -version

Vous devriez voir : openjdk version "17.x.x"

**\[votre-user\]** - Configurez JAVA_HOME :

echo 'export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"' | sudo tee -a ~/.bashrc

echo 'export PATH=\$PATH:\$JAVA_HOME/bin' | sudo tee -a ~/.bashrc

source ~/.bashrc

**\[votre-user\]** - Vérifiez JAVA_HOME :

echo \$JAVA_HOME

Devrait afficher : /usr/lib/jvm/java-17-openjdk-amd64

- Installer maven 3.9.11

**\[votre-user\]** - Télécharger la dernière version :

cd /tmp

wget <https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz>

sudo tar xzf apache-maven-3.9.11-bin.tar.gz -C /opt

sudo ln -s /opt/apache-maven-3.9.11 /opt/maven

**\[votre-user\]** - Configurez Maven :

echo 'export M2_HOME=/opt/maven' | sudo tee -a ~/.bashrc

echo 'export PATH=\$PATH:\$M2_HOME/bin' | sudo tee -a ~/.bashrc

source ~/.bashrc

**\[votre-user\]** - Vérifiez Maven :

mvn -version

Devrait afficher : Apache Maven 3.9.11

- Installation de Ant

\[votre-user\] - Installez Ant :

sudo apt install -y ant

\[votre-user\] - Vérifiez Ant :

ant -version

- Installer PostgreSQL (Client & Serveur) :

\[votre-user\] - Installez PostgreSQL :

sudo apt install -y postgresql postgresql-contrib

\[votre-user\] - Démarrez PostgreSQL :

sudo systemctl start postgresql

sudo systemctl enable postgresql

\[votre-user\] - Vérifiez le statut :

sudo systemctl status postgresql

Devrait afficher : active (running)

- Configure DSpace database, pgcrypto extension in PostgreSQL:

6.1 Configuration de la base de données

\[votre-user\] - Passez à l'utilisateur postgres :

sudo -i -u postgres

**\[postgres\]** - Vous êtes maintenant connecté en tant que postgres. Créez la base :

psql

**\[postgres via psql\]** - Dans le shell PostgreSQL, exécutez :

sql

_\-- Créer l'utilisateur dspace avec un mot de passe fort_

CREATE USER dspace WITH PASSWORD 'DSpace2024_CDNSS!Secure';

_\-- Créer la base de données_

CREATE DATABASE dspace OWNER dspace ENCODING 'UTF8';

_\-- Donner tous les privilèges_

GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;

_\-- Se connecter à la base dspace_

\\c dspace

_\-- Activer l'extension pgcrypto (requis par DSpace)_

CREATE EXTENSION IF NOT EXISTS pgcrypto;

_\-- Donner les droits sur le schéma public_

GRANT ALL ON SCHEMA public TO dspace;

_\-- Quitter psql_

\\q

**\[postgres\]** - Retournez à votre utilisateur normal :

exit

**\[votre-user\]** - Vous êtes de retour en tant que votre utilisateur normal.

6.2 Configuration de l'accès PostgreSQL

**\[votre-user\]** - Éditez le fichier pg_hba.conf :

sudo nano /etc/postgresql/14/main/pg_hba.conf

\*\*\[votre-user\]\*\* - Ajoutez ces lignes AVANT la ligne \`local all all peer\` :

_\# DSpace local connections_

local dspace dspace md5

host dspace dspace 127.0.0.1/32 md5

host dspace dspace ::1/128 md5

**\[votre-user\]** - Sauvegardez et quittez (ESC, puis :wq)

**\[votre-user\]** - Optimisez PostgreSQL pour DSpace :

sudo nano /etc/postgresql/14/main/postgresql.conf

**\[votre-user\]** - Trouvez et modifiez ces lignes (décommentez si nécessaire) :

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

Entrez le mot de passe : DSpace2024_CDNSS!Secure Si connecté avec succès, tapez \\q pour quitter

- Installation d'Apache Solr 9.x

**\[votre-user\]** - Téléchargez Solr 9.10.0 (compatible DSpace 9.1) :

cd /tmp

wget -c <https://www.apache.org/dyn/closer.lua/solr/solr/9.10.0/solr-9.10.0.tgz?action=download>

**\[votre-user\]** \- Move the tar to rename it

mv solr-9.10.0.tgz\\?action\\=download solr-9.10.0.tgz

**\[votre-user\]** - Extrayez Solr :

sudo tar xzf solr-9.10.0.tgz -C /opt

sudo mv /opt/solr-9.10.0 /opt/solr

- Création de l'Utilisateur Système DSpace

\[votre-user\] - Créez l'utilisateur dspace :

sudo useradd -m -s /bin/bash dspace

\[votre-user\] - Définissez un mot de passe :

sudo passwd dspace

Entrez un mot de passe sécurisé (ex: DSpaceUser2024!)

\[votre-user\] - Donnez les droits sudo à l'utilisateur dspace (optionnel, pour faciliter l'administration) :

sudo usermod -aG sudo dspace

\[votre-user\] - Donnez la propriété de Solr à l'utilisateur dspace :

sudo chown -R dspace:dspace /opt/solr

- Téléchargement et Compilation de DSpace 9.4.1

\[votre-user\] - Passez à l'utilisateur dspace :

sudo su - dspace

\[dspace\] - Vous êtes maintenant connecté en tant qu'utilisateur dspace.

sudo nano /home/dspace/.bashrc

\[dspace\] - Collez ceci a la fin du document :

export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"

export M2_HOME="/opt/maven"

export PATH="\$PATH:\$JAVA_HOME/bin:\$M2_HOME/bin"

\[dspace\] - Mettez a jour l'environement :

source ~/.bashrc

Téléchargez DSpace :

cd ~

git clone <https://github.com/DSpace/DSpace.git> dspace-source

cd dspace-source

\[dspace\] - Basculez sur la version 9.1 :

git checkout dspace-9.1

\[dspace\] - Vérifiez que vous êtes sur la bonne version :

git branch

Devrait afficher : \* dspace-9.1

9.1 Configuration de DSpace

\[dspace\] - Éditez le fichier de configuration :

nano dspace/config/local.cfg

\[dspace\] - collez le contenu du fichier « local.cfg » dans le dossier de configuration/installation

**\[dspace\]** - Sauvegardez et quittez

9.2 Configuration des Schémas de Métadonnées CDNSS

**\[dspace\]** - Créez le fichier de schéma personnalisé :

mkdir -p /home/dspace/dspace-source/dspace/config/registries

nano /home/dspace/dspace-source/dspace/config/registries/cdnss-types.xml

**\[dspace\]** - Collez le contenu du fichier « cdnss-types.xml » dans le dossier de configuration/installation

**\[dspace\]** - Sauvegardez et quittez

9.3 Compilation de DSpace

**\[dspace\]** - Compilez DSpace (cette étape peut prendre 10-20 minutes) :

cd /home/dspace/dspace-source

mvn clean package

**ATTENDEZ** que la compilation se termine. Vous devriez voir BUILD SUCCESS à la fin.

9.4 Installation de DSpace

**\[dspace\]** - Créez le répertoire d'installation :

sudo mkdir -p /dspace

sudo chown dspace:dspace /dspace

**\[dspace\]** - Installez DSpace :

cd /home/dspace/dspace-source/dspace/target/dspace-installer

ant fresh_install

**ATTENDEZ** que l'installation se termine. Vous devriez voir BUILD SUCCESSFUL.

**\[dspace\]** - Vérifiez que l'installation s'est bien passée :

ls -la /dspace

Vous devriez voir les dossiers: assetstore, bin, config, lib, solr, webapps, etc.

**9.5 Chargement du Schéma de Métadonnées CDNSS**

**\[dspace\]** - Chargez le schéma personnalisé :

/dspace/bin/dspace registry-loader -m /home/dspace/dspace-source/dspace/config/registries/cdnss-types.xml

- Creation de l'Administrateur

Création de l'Administrateur

\[dspace\] - Créez l'utilisateur administrateur initial :

/dspace/bin/dspace create-administrator

\`\`\`

\*\*\[dspace\]\*\* - Répondez aux questions :

\`\`\`

E-mail address: <admin@pheoc.cm>

First name: Admin

Last name: CDNSS

Language (fr, en, etc.): fr

Password: \[entrez un mot de passe sécurisé, ex: AdminCDNSS2024!\]

Retype password: \[répétez le mot de passe\]

Vous devriez voir: Administrator account created

- Configuration et Démarrage de Solr

\[dspace\] - Copiez les configurations Solr de DSpace :

cp -r /dspace/solr/\* /opt/solr/server/solr/

\[dspace\] - Démarrez Solr :

/opt/solr/bin/solr start

Vous devriez voir: Started Solr server on port 8983

\[dspace\] - Vérifiez que Solr fonctionne :

curl <http://localhost:8983/solr/admin/cores?action=STATUS>

**(if errors occur, you need to solve libraries errors:  
cp /opt/solr/modules/analysis-extras/lib/lucene-analysis-icu-9.12.3.jar /opt/solr/server/solr-webapp/webapp/WEB-INF/lib/**

**cp /opt/solr/modules/analysis-extras/lib/icu4j-\*.jar /opt/solr/server/solr-webapp/webapp/WEB-INF/lib/**

**)**

Vous devriez voir une réponse JSON avec les cores Solr

**NB: IF THE CORES ARE NOT CREATED AUTOMATICALLY, YOU CAN CREATE THEM MANUALLY: (Check CORES EXAMPLE FILES in directory)**

\[dspace\] - Créez les cores Solr pour DSpace 8.1 :

bash/opt/solr/bin/solr create -c authority -d /dspace/solr/authority

/opt/solr/bin/solr create -c oai -d /dspace/solr/oai

/opt/solr/bin/solr create -c search -d /dspace/solr/search

/opt/solr/bin/solr create -c statistics -d /dspace/solr/statistics

Pour chaque core, vous devriez voir: Created new core 'xxx'

\[dspace\] - Vérifiez que tous les cores sont créés :

curl -s "<http://localhost:8983/solr/admin/cores?action=STATUS>" | grep -o '"name":"\[^"\]\*"'

Vous devriez voir: **authority, oai, qaevent, search, statistics, suggestion**

- Installation et Configuration de Tomcat 10

**\[dspace\]** - Retournez à votre utilisateur normal :

exit

**\[votre-user\]** - Vous êtes maintenant de retour en tant que votre utilisateur normal. Téléchargez Tomcat 9 :

bash

cd /tmp

wget <https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.49/bin/apache-tomcat-10.1.49.tar.gz>

**\[votre-user\]** - Installez Tomcat :

sudo tar zxf apache-tomcat-10.1.49.tar.gz -C /opt

sudo mv /opt/apache-tomcat-10.1.49 /opt/tomcat

sudo chown -R dspace:dspace /opt/tomcat

**12.1 Configuration de Tomcat pour DSpace**

**\[votre-user\]** - Configurez la mémoire Java pour Tomcat :

sudo nano /opt/tomcat/bin/setenv.sh

**\[votre-user\]** - Ajoutez ces lignes :

# !/bin/bash

\# Configuration mémoire pour DSpace 8.1

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

export CATALINA_OPTS="-Xmx2048m -Xms1024m -Dfile.encoding=UTF-8 -Djava.awt.headless=true"

export JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8"

**\[votre-user\]** - Rendez le fichier exécutable :

sudo chmod +x /opt/tomcat/bin/setenv.sh

**12.2 Configuration du connecteur HTTP**

**\[votre-user\]** - Éditez server.xml :

sudo nano /opt/tomcat/conf/server.xml

**\[votre-user\]** - Trouvez la section <Connector port="8080" et remplacez-la par :

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

**12.3 Déploiement de l'application DSpace**

**\[votre-user\]** - create a symbolic link from Tomcat to your DSpace installation « server » folder :

sudo ln -s /dspace/webapps/server /opt/tomcat/webapps/server

sudo chown dspace:dspace /opt/tomcat/webapps/server

**12.4 Création du service systemd pour Tomcat**

**\[votre-user\]** - Créez le fichier service :

sudo nano /etc/systemd/system/tomcat.service

**\[votre-user\]** - Ajoutez ce contenu :

\[Unit\]

Description=Apache Tomcat pour DSpace 9.1

After=network.target postgresql.service

\[Service\]

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

\[Install\]

WantedBy=multi-user.target

**\[votre-user\]** - Sauvegardez et quittez

**\[votre-user\]** - Rechargez systemd :

sudo systemctl daemon-reload

**\[votre-user\]** - Activez le service Tomcat :

sudo systemctl enable tomcat

**\[votre-user\]** - Démarrez Tomcat :

sudo systemctl start tomcat

**\[votre-user\]** - Vérifiez le statut :

sudo systemctl status tomcat

Vous devriez voir active (running)

**\[votre-user\]** - Attendez 30-60 secondes que l'application se déploie, puis vérifiez les logs :

sudo tail -f /opt/tomcat/logs/catalina.out

Appuyez sur CTRL+C pour arrêter de suivre les logs Cherchez le message: Server startup in \[xxxx\] milliseconds

- Indexation Initiale avec Solr

\[votre-user\] - Passez à l'utilisateur dspace :

sudo su - dspace

\[dspace\] - Créez l'index de recherche initial :

/dspace/bin/dspace index-discovery -b

**(IF THERE ARE JAVA OR OTHER MODULES ERRORS DUE TO ENVIRONMENT, RUN THIS:  
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin)**

Cette commande peut prendre quelques minutes. Vous devriez voir des messages de progression.

THE DSPACE BACKEND IS UP, RUNNING AND FULLY FUNCTIONNAL AT THIS POINT.

Now we have to customize the backend server api's, indexes etc

**PHASE 2 : CUSTOMISATION ET CONFIGURATION DE DSPACE**
