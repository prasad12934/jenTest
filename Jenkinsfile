pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        // ✅ Pulls clientId as _USR and clientSecret as _PSW from Jenkins credential
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
                bat 'mvn clean package -U -DskipTests'
            }
        }

        // ✅ Publish to Exchange FIRST — no settings.xml needed
        // Connected App credentials passed via -Danypoint.username / -Danypoint.password
        stage('Publish to Exchange') {
            steps {
                bat """
                    mvn deploy -DskipTests ^
                    -Danypoint.username=~~~Client~~~%ANYPOINT_CLIENT_USR% ^
                    -Danypoint.password=%ANYPOINT_CLIENT_PSW%
                """
            }
        }

        // ✅ Deploy to CloudHub 2.0
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
            echo '🚀 Deployment SUCCESS'
        }
        failure {
            echo '❌ Deployment FAILED'
        }
    }
}