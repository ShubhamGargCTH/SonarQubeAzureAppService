# Sonar Qube as Azure app service docker container

All Sonar qube docker container post 7.7 release throws error when trying to host Sonar Quber as azure app service.

Error - max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

This error is thrown by the elastic search cluster. As of now there is no way to increase this limit in azure app service. 

This check is part of elastic search bootstrap checks. From the elastic search docs I found that it disables the mandatory bootstrap checks when hosted on single node or with local address. So it should ideally work in azure app service since elstic search is hosted as local address but it doesn''t.

Going through the sonar quobe source code i found out that sonar quebe enforces bootstrap check when its not running embedded h2 database.

See line -144 here  - https://github.com/SonarSource/sonarqube/blob/3f6f5496277c76c2498fd245a112931d19830497/server/sonar-main/src/main/java/org/sonar/application/command/EsJvmOptions.java

This code also tells us that we can disable the bootstrap checks by setting env variable - sonar.es.bootstrap.checks.disable to true. Simple enough but doesnt work.

After hours of research i found out that azure app service replace "." with "-" for the values passed as appsetting. Thanks to post here - https://r3dlin3.github.io/2020/01/01/sonarqube-app-service/

I tried the option given by author in above post but that still doesn't work.

Finally as a work around in the run.sh i added the jvm option sonar.es.bootstrap.checks.disable directly. Notice line 43 in the run.sh file.

And whoo finally my Sonar Qube is up and running in azure app service.

Steps- 
1. Download all three files in a folder
2. Create the docker image
3. Push the docker images
4. Run
 
 I will add the commands to all these steps also i will submit a PR in Sonar Qube repo so that we can pass sonar.es.bootstrap.checks.disable as env variables from azure app service.
