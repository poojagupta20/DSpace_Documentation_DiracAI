# DSpace_Installation in Ubuntu 22.04lts

 One need to install maven 3.X , Java-17.X , tomcat 10.X, solr 8.11.X, node 20.x and ant 1.X

 ## Instructions: Database and environment 
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

## Instructions: Servers tomcat and solr
