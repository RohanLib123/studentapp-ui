pipeline {
    agent any

    tools {
        jdk 'jdk-17'   // or jdk-17
        maven 'maven-3.9'
    }
    
    environment {
        APP_NAME = "my-java-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/RohanLib123/studentapp-ui.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.{jar,war}'

            }
        }
    }

    post {
        success {
            echo "Build successful 🚀"
        }
        failure {
            echo "Build failed ❌"
        }
    }
}
