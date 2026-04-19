pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    environment {
        ANYPOINT_CLIENT = credentials('anypoint-connected-app')
        // Path where we'll write the temporary settings.xml
        MAVEN_SETTINGS = "${WORKSPACE}\\settings.xml"
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

        // ✅ Dynamically create settings.xml at runtime — no manual file needed
        stage('Generate Maven Settings') {
            steps {
                script {
                    def settingsContent = """<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>anypoint-exchange-v3</id>
            <username>~~~Client~~~${env.ANYPOINT_CLIENT_USR}</username>
            <password>${env.ANYPOINT_CLIENT_PSW}</password>
        </server>
    </servers>
</settings>"""
                    writeFile file: 'settings.xml', text: settingsContent
                    echo "✅ settings.xml generated at ${WORKSPACE}\\settings.xml"
                }
            }
        }

        // ✅ Publish to Exchange using the dynamically generated settings.xml
        stage('Publish to Exchange') {
            steps {
                bat """
                    mvn deploy -DskipTests ^
                    --settings %WORKSPACE%\\settings.xml
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
        always {
            // ✅ Clean up the generated settings.xml so secrets don't stay on disk
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