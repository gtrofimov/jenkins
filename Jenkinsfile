pipeline {
    agent any
    options {
        // This is required if you want to clean before build
        skipDefaultCheckout(true)
    }
    environment {
        // app_git_url='https://github.com/parasoft/parabank.git'
        // app_branch='master'
        // app_repo=''
        // build_repo=''
        // test_repo=''

        parabank_port=8090
        ls_url="${PARASOFT_LS_URL}"
        ls_user="${PARASOFT_LS_USER}"
        ls_pass="${PARASOFT_LS_PASS}"

    }
    stages {
        stage('Clone Repo') {
            steps {
                cleanWs()
                sh '''
                    echo ${ls_user}
                    ls -R    
                    
                    git clone -b tia 'https://github.com/gtrofimov/jenkins.git' 
                
                    git clone -b selenium-demo 'https://github.com/parasoft/parabank.git'
                    ls -la
                
                   '''
            }
        }
        stage('Build App with Jtest SA') {
            when { equals expected: true, actual: true}
            steps {
                sh '''
                
                echo "Starting Pipeline Execution."
                echo ${PWD}

                # Check Vars
                echo $PWD
                mkdir monitor
                # set MONITOR_HOME="monitor"
                # is Docker runnnig?
                docker ps
                # docker network create ${JOB_NAME} 
                ls -la

                # Build with Jtest SA/UT/monitor
                # Set Up and write .properties file

                echo  -e "\n~~~\nSetting up and creating jtest.properties file.\n"
                echo $"
                parasoft.eula.accepted=true
                jtest.license.use_network=true
                jtest.license.network.edition=server_edition
                dtp.url=${ls_url}
                dtp.user=${ls_user}
                dtp.password=${ls_pass}" >> jenkins/jtest/jtestcli.properties
                echo -e "\nDebug -- Verify workspace contents.\n"
                ls -la
                echo -e "\nDebug -- Verify jtestcli.properties file contents."
                cat jenkins/jtest/jtestcli.properties

                # need to point to ${user.home}
                
                docker run --rm -i \
                -u 0:0 \
                -v "$PWD:$PWD" \
                -w "$PWD" \
                $(docker build -q ./jenkins/jtest) /bin/bash -c " \
                cd parabank; \
                mvn \
                -Dmaven.test.failure.ignore=true \
                test-compile jtest:agent \
                test jtest:jtest \
                -s /home/parasoft/.m2/settings.xml \
                -Djtest.settings='/home/parasoft/jtestcli.properties' \
                -Djtest.config='jtest.dtp://UTSA'; \
                mvn \
                -DskipTests=true \
                package jtest:monitor \
                -s /home/parasoft/.m2/settings.xml \
                -Djtest.settings='/home/parasoft/jtestcli.properties'; \
                "

                # Unzip monitor.zip
                unzip **/target/*/*/monitor.zip -d .
                ls -la monitor
                '''
            }
        }
        stage('Deploy App via Docker') {
            when { equals expected: true, actual: true}
            steps {
                sh '''
                echo ${PWD}
                docker ps

                # Set Up Env
                # Unpack monitor
                # JDBCDriver.jar

                # Deploy App
                ## Specify Port via ${parabank_port}
                docker run -d \
                -p ${parabank_port}:8080 \
                -p 8050:8050 \
                -p 61616:61616 \
                -p 9001:9001 \
                --env-file "$PWD/jenkins/jtest/monitor.env" \
                -v "$PWD/monitor:/home/docker/jtest/monitor" \
                --name parabankv1 \
                parasoft/parabank

                # Configure App

                # Deploy Virt Env
                '''
            }
        }
        stage('Run SOAtest Tests') {
            when { equals expected: true, actual: true}
            steps {
                sh '''
                echo ${pwd}
                
                # Wait for Parabank start up
                # sleep 30
                docker ps

                # Pulse?
                # curl -iv --raw http://localhost:8090/parabank
                # curl -iv --raw http://localhost:8050/status
                
                # License SOAtest
                # Set Up and write .properties file
                echo  -e "\n~~~\nSetting up and creating soatestcli.properties file.\n"
                echo $"
                license.network.auth.enabled=true
                license.network.use.specified.server=true
                license.network.url=${ls_url}
                license.network.user=${ls_user}
                license.network.password=${ls_pass}
                soatest.license.network.edition=automation_edition
                soatest.license.use_network=true" >> jenkins/soatest/soatestcli.properties
                echo -e "\nDebug -- Verify workspace contents.\n"
                ls -la jenkins/soatest
                echo -e "\nDebug -- Verify soatestcli.properties file contents."
                cat jenkins/soatest/soatestcli.properties

                # Run SOAtest Tests
                
                docker run --rm -i \
                -u 0:0 \
                -e ACCEPT_EULA=true \
                -v "$PWD:$PWD" \
                parasoft/soavirt /bin/bash -c " \
                cat $PWD/jenkins/soatest/soatestcli.properties; \
                soatestcli \
                -settings $PWD/jenkins/soatest/soatestcli.properties \
                -machineId; \
                ls -la $PWD/jenkins/soatest; \
                cp "$PWD/jenkins/soatest"/* "/root/parasoft/soavirt_workspace/TestAssets/"; \
                soatestcli \
                -resource /TestAssets \
                -config '/root/parasoft/soavirt_workspace/TestAssets/AppCoverage.properties' \
                -settings $PWD/jenkins/soatest/soatestcli.properties \
                -environment 127.17.0.1 \
                -report $PWD/jenkins/soatest/report"
                
                '''


            }
        }
        stage('Destroy Contatiners and Clean Up') {
            when { equals expected: true, actual: true}
            steps {
                sh '''
                echo ${pwd}
                docker ps
                docker stop parabankv1
                docker rm parabankv1
                '''
            }
        }
        stage('Static Analysis Reports'){
            when { equals expected: true, actual: false}
            steps {
                echo '---> Parsing static analysis reports'
                step([$class: 'ParasoftPublisher', useReportPattern: true, reportPattern: '**/target/jtest/*.xml', settings: ''])      
            }
            
        
        }
        stage('Unit Test 10x'){
            steps {
                echo '---> Parsing 10.x unit test reports'
                step([$class: 'XUnitPublisher', 
                    tools: [
                        [$class: 'ParasoftType', 
                            pattern: '**/target/jtest/*.xml', 
                            failIfNotNew: false, 
                            skipNoTestFiles: true, 
                            stopProcessingIfError: false
                        ]
                    ]
                ])
            }
        }
        stage('Functional Tests 9x'){
            steps {
                echo '---> Parsing 9.x functional test reports'
                step([$class: 'XUnitPublisher', 
                    tools: [
                        [$class: 'ParasoftSOAtest9xType', 
                            pattern: '**/soatest/report/*.xml', 
                            failIfNotNew: false, 
                            skipNoTestFiles: true, 
                            stopProcessingIfError: false
                        ]
                    ]
                ])
            }
        }
    }

    post {
        always {
            
            archiveArtifacts artifacts: '**/target/jtest/**, **/soatest/report/**',
                excludes: '**/cache/**',
                fingerprint: true, 
                onlyIfSuccessful: true
        }
    }
}