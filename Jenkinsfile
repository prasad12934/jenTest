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
                bat 'mvn clean package -U -DskipTests'
            }
        }

        // ✅ Write settings.xml using bat/echo — avoids Groovy string interpolation security issue
        stage('Generate Maven Settings') {
            steps {
                bat '''
                    echo ^<?xml version="1.0" encoding="UTF-8"?^> > settings.xml
                    echo ^<settings^> >> settings.xml
                    echo   ^<servers^> >> settings.xml
                    echo     ^<server^> >> settings.xml
                    echo       ^<id^>anypoint-exchange-v3^</id^> >> settings.xml
                    echo       ^<username^>~~~Client~~~%ANYPOINT_CLIENT_USR%^</username^> >> settings.xml
                    echo       ^<password^>%ANYPOINT_CLIENT_PSW%^</password^> >> settings.xml
                    echo     ^</server^> >> settings.xml
                    echo   ^</servers^> >> settings.xml
                    echo ^</settings^> >> settings.xml
                    echo Settings file written successfully.
                '''
            }
        }

        stage('Publish to Exchange') {
            steps {
                bat 'mvn deploy -DskipTests --settings %WORKSPACE%\\settings.xml'
            }
        }

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
        always {
            bat 'if exist "%WORKSPACE%\\settings.xml" del "%WORKSPACE%\\settings.xml"'
        }
        success {
            echo '🚀 Deployment SUCCESS'
        }
        failure {
            echo '❌ Deployment FAILED'
        }
    }
}