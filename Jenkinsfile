pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven'
    }
    
    environment {
    SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/KPgit-5/EKart.git'
            }
        }
        
        stage('Compile') {
            steps {
                 sh "mvn clean compile"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                 withSonarQubeEnv('SonarQube-Server') {
          sh '''
            $SCANNER_HOME/bin/sonar-scanner \
              -Dsonar.projectKey=Ekart \
              -Dsonar.projectName=Ekart \
              -Dsonar.sources=. \
              -Dsonar.java.binaries=. \
              -Dsonar.host.url=$SONAR_HOST_URL \
              -Dsonar.login=$SONAR_AUTH_TOKEN
          '''
                }
            }
        }
    }
}
