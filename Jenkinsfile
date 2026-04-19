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

        // ❌ REMOVED: Publish to Exchange (THIS WAS CAUSING 401)

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