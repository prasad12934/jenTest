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

        // ✅ STEP 1: PUBLISH TO EXCHANGE (CRITICAL)
        stage('Publish to Exchange') {
            steps {
                bat """
                    mvn clean deploy ^
                    -DskipTests ^
                    -Dconnected.app.clientId=%ANYPOINT_CLIENT_USR% ^
                    -Dconnected.app.clientSecret=%ANYPOINT_CLIENT_PSW%
                """
            }
        }

        // ✅ STEP 2: DEPLOY FROM EXCHANGE
        stage('Deploy to CloudHub') {
            steps {
                bat """
                    mvn mule:deploy ^
                    -DmuleDeploy ^
                    -DskipTests ^
                    -Dconnected.app.clientId=%ANYPOINT_CLIENT_USR% ^
                    -Dconnected.app.clientSecret=%ANYPOINT_CLIENT_PSW%
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