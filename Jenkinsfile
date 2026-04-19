pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        ANYPOINT_CLIENT = credentials('anypoint-connected-app')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to CloudHub') {
            steps {
                bat """
                    mvn mule:deploy ^
                    -Dconnected.app.clientId=%ANYPOINT_CLIENT_USR% ^
                    -Dconnected.app.clientSecret=%ANYPOINT_CLIENT_PSW% ^
                    -DskipTests
                """
            }
        }
    }

    post {
        success {
            echo '🚀 CloudHub Deployment SUCCESS'
        }
        failure {
            echo '❌ Deployment FAILED'
        }
    }
}