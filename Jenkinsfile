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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectKey=Ekart \
                          -Dsonar.projectName=Ekart \
                          -Dsonar.sources=. \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan . --format HTML,XML --project "Ekart" --out odc-report --failOnCVSS 7', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: 'odc-report/dependency-check-report.xml'
            }
        }

        stage('Build Application') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '3bdcb0cd-9708-4e7e-a3f9-3e6aee2d6cfc') {
                        sh "docker build -t Ekart:latest -f docker/Dockerfile ."
                        sh "docker tag Ekart:latest Kalpanapandey/Ekart:latest"
                        sh "docker push Kalpanapandey/Ekart:latest"
                    }
                }
            }
        }

        stage('Trigger CD-pipeline') {
            steps {
                build job: "CD-pipeline", wait: true
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'odc-report/dependency-check-report.html', allowEmptyArchive: true
        }
    }
}
