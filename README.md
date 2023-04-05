# Parabank build/deploy/test/promote
The followig repo conatins automation script for Jenkins to run a full build of Parabank with Jtest/SOAtest/Virtualize/otehr tool vi ephemaral Docker conatiners

## Project Contents

```
jenkins
 ┣ jtest
 ┃ ┣ .m2
 ┃ ┃ ┗ settings.xml
 ┃ ┣ Dockerfile
 ┃ ┗ jtestcli.properties
 ┣ .gitignore
 ┣ Jenkinsfile
 ┗ README.md
```
## To Run

1. Install Jenkins on a Linux VM (https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/#downloading-and-installing-jenkins)
   1. set jenkis vars
2. Install Docker (link to Docker install on Linux)
3, Install Parasoft DTP
   1. add configs
4. Create Jenkins Pipeline
   1. Build
   2. Deploy
   3. Test
   4. Release
5. Configure licenses for each tool
   1. jtest
   2. virt
   3. soatest
   4. well whats parabank missing really
### Potential Errors and Workarounds (FAQ)
1. add sudouser
2. tty (unresloved)
   
TODO: Add ADO, Terraform

## Stages

### Test

#### Jtest

To Jtest to run configure `jtest/jtestcli.properties` file.

The Dockerfile will create a Jtest docker container with Maven installed. The settings.xml file and `jtestcli.properties` file is copied into the `/home/parasoft/` directory.

to run jtestcli use
```
mvn jtest:jtest \
-s /home/parasoft/.m2/settings.xml \
-Djtest.settings='/home/parasoft/jtestcli.properties'; \
```

TODO:
fix docker file so that the -s and -Djtest.settings flags arent needed.

### Deploy
#### Parabank
#### Virtulize
### Test
#### SOAtest
#### Selenic
### Release