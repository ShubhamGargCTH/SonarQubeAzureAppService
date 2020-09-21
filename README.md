# Hosting SonarQube IN AZURE APP SERVICE AND AZURE SQL DATABASE

This is a walk through for hosting SonarQube community version in azure.

*Prerequisite*
- Knowledge of running contanarized apps in Azure App Service
- Knowledge Azure SQL Db

For this walkthrough we will be using SonarQube 7.9(LTS) and SonarQube 8.4.2.36762(Latest at the time of writing)

If you have not tried already, When  Official SQ docker image for any version >7.7 is hosted on azure with a persistant database it fails with *max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]* error. This error is thrown by the elastic search cluster. As of now there is no way to increase this limit in azure app service. Since its a PAAS offering and user don't have a way to access underlying azure vm.

This error is thrown from the elastic search bootstrap checks. We can disable this check some of the times. As elastic search docs states that bootstrap checks are not mandatory if running inside a local address or single node. So it should ideally work in azure app service since elstic search is hosted as local address but it doesn't. Going through the sonar quobe source code i found out that sonar quebe enforces bootstrap check when its not running embedded h2 database.

See line -144 here  - https://github.com/SonarSource/sonarqube/blob/3f6f5496277c76c2498fd245a112931d19830497/server/sonar-main/src/main/java/org/sonar/application/command/EsJvmOptions.java

As per above code we should be able to set env variabele #sonar.es.bootstrap.checks.disable to true and SQ won't enforce bootstrap checks.
Here is another issue. azure app service replace "." with "-" for the values passed as appsetting.  So we won't really be able to set the env variable untill and unless SQ changes the env variable name. Good news is SQ docker images are available on git hub and we can modify these images to suite our needs.

Lets start by creating the Azure SQL Database
1. Create a new azure sql server resource
2. On Networking tab make sure to allow azure service to use this server
3. Create a new Azure sql database
4. Set the collation as *SQL_Latin1_General_CP1_CS_AS*
5. Copy the jdbc connection string

Now lets create our own image for SQ 7.9 version
 1. Clone/Download  sonar-7 folder from this git hub repository. Only change is in run.sh file notice line 30 here i am passing -Dsonar.es.bootstrap.checks.disable="$SONAR_ES_BOOTSTRAP_CHECKS_DISABLE".
 2. Build the docker image
 3. Push to registry
 4. Create app service and use the image just created
 5. Set the env variables
     - SONARQUBE_JDBC_URL = <copied jdbc url> (make sure to replace the password)
     - SONARQUBE_JDBC_PASSWORD = <sql server password>
     - SONARQUBE_JDBC_USERNAME = <sql server password>
     - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE = true
     - WEBSITES_CONTAINER_START_TIME_LIMIT = 400 (Optional this is to override default wait time limit of container start)
 With some luck you should be able to browse to SonarQube instance. :) 
 
 #For Latest 8.4 version 
 1. download the DockerFile,run.sh and sonar.sh file from the root directory of repo.
 2. Build the docker image. Notice line 43 in run.sh. Thats the only difference in offical image and this image.
 3. Push to registry
 4. Create app service and use the image just created
 5. Set the env variables
     - SONAR_JDBC_URL = <copied jdbc url> (make sure to replace the password)
     - SONAR_JDBC_PASSWORD = <sql server password>
     - SONAR_JDBC_USERNAME = <sql server password>
     - WEBSITES_CONTAINER_START_TIME_LIMIT = 400
 No need to set the env variable to disable the check as we are hardcoding directly in run.sh. If you want you can pass it as env variable as well.
 
 Thats all fols you should now be able to run SQ with Azure App Service and Azure SQL Database,
