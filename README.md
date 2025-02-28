# DSpace_Installation in Ubuntu 22.04lts

 One need to install maven 3.X , Java-17.X , tomcat 10.X, solr 8.11.X, node 20.x and ant 1.X

 ## Directory hierarchy
   -  go to any area inside your laptop 
   -  `mkdir dspace`
   -  `cd dspace`
   -  `mkdir root` # will be used for compiled files
   -  `mkdir servers` # will be used for tomcat and solr installation
   -  `mkdir source`  # will be used for front-end and back-end installatio

 ## Instructions: prerequisites and database 
 -   `sudo apt update && sudo apt install ant -y`
 -   `sudo apt install maven`
 -   `sudo apt install openjdk-17-jdk`
 -   `sudo apt install  git unzip wget curl -y`
 -   `sudo apt install postgresql postgresql-contrib -y`  # version should 12 , 13 or 14 as on date 28th Feb 2025
 -   `sudo -i -u postgres psql`
 -   `CREATE DATABASE dspace;`
   
     `CREATE USER dspace WITH PASSWORD 'dspace';`
    
     `ALTER DATABASE dspace OWNER TO dspace;`
    
     `GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;`

## Installation of  servers and search engine :  tomcat and solr
-   Apache Solr is an open-source search engine and enterprise search platform. Apache Tomcat is a free, open-source web server and servlet container that's used to host Java-based web applications.
-   dspace code expects some files from solr and tomcat to be present , so we need to download and untar them first
-   `cd servers/`

-   `wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.36/bin/apache-tomcat-10.1.36.tar.gz`
-   `tar -xvzf apache-tomcat-10.1.36.tar.gz`

-   `wget https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.4/solr-8.11.4.tgz?action=download -O solr-8.11.4.tgz`
-   `tar -xvzf solr-8.11.4.tgz`

-   `rm -rf solr-8.11.4.tgz apache-tomcat-10.1.36.tar.gz`




## Installation of dspace  Frontend and Backend codes

-   `cd dspace/source`
  
-   `git clone https://github.com/DSpace/DSpace.git backend`
  
-   `git clone https://github.com/DSpace/dspace-angular.git frontend`
  
-   `touch backend/dspace/config/local.cfg`
  
-   `cp backend/dspace/config/local.cfg.EXAMPLE backend/dspace/config/local.cfg`
  
-    Now you need to go to the root folder and print the path of root with the command `pwd`. It will print something like `/home/bibhu/DiracAI-Services/DSpace/root` . Now put this path inside local.cfg  like `dspace.dir=/home/bibhu/DiracAI-Services/DSpace/root`

## Backend setup

-     now go to dspace/source/backend
-     cd backend
-    `mvn clean package`
-     after build success
-    `cd dspace/target/dspace-installer`
-    `ant fresh_install`
-     after build success

-     now check wheather the root folder does it have any folders in it or not

-     now got to dspace/root/bin

-    `cd ../../../../../root/bin`
-    `sudo ./dspace database migrate`
-    `sudo ./dspace create-administrator`
-     give username = admin@gmail.com
-     firstname = admin
-     lastname = admin
-     select y
-     password as admin
-     reenter the password admin

-     now copy the generated solr files in root folder to our solr server configsets folder by deleting the existing files in server solar files

-     `rm -rf ../../servers/solr-8.11.4/server/solr/configsets/`

-     `cp -rf ../solr/* ../../servers/solr-8.11.4/server/solr/configsets`

-     `cp -rf ../webapps/server ../../servers/apache-tomcat-10.1.36/webapps`

-     `cd  ../../servers/solr-8.11.4/bin/'

-      `sudo ./solr start -p 8983 -force`

-     `cd  ../../apache-tomcat-10.1.36/bin`

-     `./startup.sh`
## Frontend setup

-  now go to frontend cloned path folder

-  `cd ../../../source/frontend`

-  `touch config/config.prod.yml`

-   `cp config/config.example.yml config/config.prod.yml`

-   Now open config.prod.yml file and look for the line something like

 `rest:
  ssl: true
  host: sandbox.dspace.org
  port: 443
  nameSpace: /server`

  and change to 

  `rest:
  ssl: false
  host: localhost
  port: 8080
  nameSpace: /server`

  - now start the frontend server
  - `npm install`
  - `npm start`

   
   
   
   



    



