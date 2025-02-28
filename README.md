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
 -   `sudo apt install maven`
 -   `sudo apt install openjdk-17-jdk`
 -   `sudo apt update && sudo apt install ant -y`
 -   `sudo apt install  git unzip wget curl -y`
 -   `sudo apt install postgresql postgresql-contrib -y`  # version should 12 , 13 or 14 as on date 28th Feb 2025
 -   `sudo -i -u postgres psql`
 -   `CREATE DATABASE dspace;`
   
     `CREATE USER dspace WITH PASSWORD 'dspace';`
    
     `ALTER DATABASE dspace OWNER TO dspace;`
    
     `GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;`

## Installation of tomcat and solr

-   `cd servers/`

-   `wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.36/bin/apache-tomcat-10.1.36.tar.gz`
-   `tar -xvzf apache-tomcat-10.1.36.tar.gz`

-   `wget https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.4/solr-8.11.4.tgz?action=download -O solr-8.11.4.tgz`
-   `tar -xvzf solr-8.11.4.tgz`

-   `rm -rf solr-8.11.4.tgz apache-tomcat-10.1.36.tar.gz`





   
   
   
   



    



