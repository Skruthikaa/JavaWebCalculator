pipeline {
    agent { label 'Sonar' }
 
    tools {
        jdk 'JDK17'
        maven 'maven'
    }
 
    environment {
        SONARQUBE_SERVER = 'sonar'
        MVN_SETTINGS = '/etc/maven/settings.xml'
        NEXUS_URL = 'http://54.234.150.233:8081'
        NEXUS_REPO = 'maven-releases'
        NEXUS_GROUP = 'com/web/cal'
        NEXUS_ARTIFACT = 'webapp-add'
        TOMCAT_URL = 'http://98.93.71.114:8080/manager/text'
    }
 
    stages {
 
        stage('Checkout Code') {
            steps {
                echo ' Cloning source from GitHub...'
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Skruthikaa/JavaWebCalculator']]
                ])
            }
        }
 
        stage('SonarQube Analysis') {
            steps {
                echo ' Running SonarQube static analysis...'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn clean verify sonar:sonar -DskipTests --settings ${MVN_SETTINGS}'
                }
            }
        }
 
        stage('Build Artifact') {
            steps {
                echo '⚙️ Building WAR...'
                sh 'mvn clean package -DskipTests --settings ${MVN_SETTINGS}'
                sh 'echo  Build Completed!'
                sh 'ls -lh target/*.war || true'
            }
        }
 
        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Nexus', usernameVariable: 'NEXUS_USR', passwordVariable: 'NEXUS_PSW')]) {
                    sh '''#!/bin/bash
                        set -e
                        WAR_FILE=$(ls target/*.war | head -1)
                        FILE_NAME=$(basename "$WAR_FILE")
                        VERSION="0.0.${BUILD_NUMBER}"
 
                        echo "📤 Uploading $FILE_NAME to Nexus as version $VERSION..."
 
                        curl -v -u ${NEXUS_USR}:${NEXUS_PSW} --upload-file "$WAR_FILE" \
                          "${NEXUS_URL}/repository/${NEXUS_REPO}/${NEXUS_GROUP}/${NEXUS_ARTIFACT}/${VERSION}/${NEXUS_ARTIFACT}-${VERSION}.war"
 
                        echo " Artifact uploaded successfully to Nexus!"
                    '''
                }
            }
        }
 
        stage('Deploy to Tomcat') {
            agent { label 'Tomcat' }
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'Nexus', usernameVariable: 'NEXUS_USR', passwordVariable: 'NEXUS_PSW'),
                    usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USR', passwordVariable: 'TOMCAT_PSW')
                ]) {
                    sh '''#!/bin/bash
                        set -e
                        cd /tmp; rm -f *.war
 
                        echo " Fetching latest WAR from Nexus..."
                        DOWNLOAD_URL=$(curl -s -u ${NEXUS_USR}:${NEXUS_PSW} \
                            "${NEXUS_URL}/service/rest/v1/search?repository=${NEXUS_REPO}&group=com.web.cal&name=webapp-add" \
                            | grep -oP '"downloadUrl"\\s*:\\s*"\\K[^"]+\\.war' | grep -vE '\\.md5|\\.sha1' | tail -1)
 
                        if [[ -z "$DOWNLOAD_URL" ]]; then
                            echo " No WAR found in Nexus!"
                            exit 1
                        fi
 
                        echo " Downloading WAR: $DOWNLOAD_URL"
                        curl -u ${NEXUS_USR}:${NEXUS_PSW} -O "$DOWNLOAD_URL"
                        WAR_FILE=$(basename "$DOWNLOAD_URL")
                        APP_NAME=$(echo "$WAR_FILE" | sed 's/-[0-9].*//')
 
                        echo " Removing old deployment..."
                        curl -u ${TOMCAT_USR}:${TOMCAT_PSW} "${TOMCAT_URL}/undeploy?path=/${APP_NAME}" || true
 
                        echo " Deploying new WAR to Tomcat..."
                        curl -u ${TOMCAT_USR}:${TOMCAT_PSW} --upload-file "$WAR_FILE" \
                            "${TOMCAT_URL}/deploy?path=/${APP_NAME}&update=true"
 
                        echo " Deployment successful! Application updated."
                    '''
                }
            }
        }
    }
 
    post {
        success { echo ' Pipeline completed successfully — Application live on Tomcat!' }
        failure { echo ' Pipeline failed — Check Jenkins logs.' }
    }
}
