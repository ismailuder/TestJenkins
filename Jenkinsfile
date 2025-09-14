pipeline {
    agent any

    environment {
        TEMP_TOOLS = "${WORKSPACE}/.temp_dotnet_tools"
        PATH = "${TEMP_TOOLS}:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Clean & Prepare') {
            steps {
                script {
                    // Önce geçici dizini temizle
                    sh 'rm -rf $TEMP_TOOLS'
                    sh 'mkdir -p $TEMP_TOOLS'

                    // Geçici dizine dotnet-sonarscanner yükle
                    sh 'dotnet tool install --tool-path $TEMP_TOOLS dotnet-sonarscanner --version 10.3.0 --ignore-failed-sources'
                }
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins/TestJenkins.csproj -c Release
                        dotnet test TestJenkins/TestJenkins.csproj -c Release
                        dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                echo "Bu adım build başarılıysa çalışır"
            }
        }
    }

    post {
        success {
            echo "Pipeline başarılı ✅"
        }
        failure {
            echo "Pipeline başarısız ❌"
        }
    }
}
