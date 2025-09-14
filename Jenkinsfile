pipeline {
    agent any

    environment {
        DOTNET_TOOLS = "/var/jenkins_home/.dotnet/tools"
        PATH = "${DOTNET_TOOLS}:${env.PATH}"
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
                    // Önce eski SonarScanner kalıntılarını temizle
                    sh 'rm -rf $DOTNET_TOOLS/dotnet-sonarscanner'
                    sh 'rm -rf $DOTNET_TOOLS/.store/dotnet-sonarscanner'

                    // SonarScanner yüklü değilse yükle
                    if (!fileExists("${DOTNET_TOOLS}/dotnet-sonarscanner")) {
                        sh 'dotnet tool install --tool-path $DOTNET_TOOLS dotnet-sonarscanner --version 10.3.0'
                    }
                }
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        dotnet sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins/TestJenkins.csproj -c Release
                        dotnet test TestJenkins/TestJenkins.csproj -c Release
                        dotnet sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                echo "Bu adım sadece build başarılıysa çalışır"
                // Burada docker build & kubectl apply komutlarını ekleyebilirsin
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
