### NEXUS REPO

##Maven conf: pom.xml

    > pour les Snapshots
    <version>1.1-SNAPSHOT</version>
    > pour les Releases
    <version>1.0</version>
    
..

On precise les repository nexus pour release et snapshot

    <distributionManagement>
        <repository>
          <id>deployment</id>
          <name>Internal Release</name>
          <url>http://localhost:8081/repository/nexusFirst-releases/</url>
        </repository>
        <snapshotRepository>
          <id>deployment</id>
          <name>Internal Snapshot</name>
          <url>http://localhost:8081/repository/nexusFirst-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
..

##CONF settings.xml
C:\Program Files\apache-maven-3.3.9\conf\
Ici on renseigne les repo present dans le pom.xml et le login pour nexus

    <server>
      <id>deployment</id>
      <username>admin</username> 
      <password>admin1234</password>
    </server>
    
##nexus

Run SimpleMode: dans la console(admin) 

      1. cd C:\Program Files (x86)\nexus-3.11.0-01\bin
      2. nexus.exe /run and wait until you see the log message ... started...
      3. And then open the url localhost:8081
      
Run ServiceMode: dans la console(admin) 

      1. cd C:\Program Files (x86)\nexus-3.11.0-01\bin
      2. nexus /install
      3. And then open the url localhost:8081
      
## DEPLOY

     mvn clean install deploy
     or
     mvn clean compile package deploy
     
     
>Ceci va deployer le projet dans nexus   