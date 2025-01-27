pipeline {
    agent any
    tools {
        maven "maven3"
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Manohar-1305/repo.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('File System scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=multi-tier -Dsonar.projectKey=multi-tier -Dsonar.java.binaries=."
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true "
                }
            }
        }
        stage('Docker Build Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t manoharshetty507/bankapp:latest ."
                    }
                }
            }
        }
        stage('Docker Image scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html manoharshetty507/bankapp:latest'
            }
        }
        stage('Docker Push') {
            steps {
                sh 'docker push manoharshetty507/bankapp:latest'
            }
        }
    }
}
