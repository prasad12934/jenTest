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

        // ✅ PowerShell writes the file using env vars directly
        // PowerShell receives the real secret value from the environment
        // even though Jenkins masks it in console output
        stage('Generate Maven Settings') {
            steps {
                powershell '''
                    $clientId     = $env:ANYPOINT_CLIENT_USR
                    $clientSecret = $env:ANYPOINT_CLIENT_PSW

                    $xml = @"
<?xml version="1.0" encoding="UTF-8"?>
<settings>
  <servers>
    <server>
      <id>anypoint-exchange-v3</id>
      <username>~~~Client~~~$clientId</username>
      <password>$clientSecret</password>
    </server>
  </servers>
</settings>
"@
                    [System.IO.File]::WriteAllText("$env:WORKSPACE\\settings.xml", $xml, [System.Text.Encoding]::UTF8)
                    Write-Host "settings.xml written successfully"
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