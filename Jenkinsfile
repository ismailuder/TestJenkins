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

        stage('Prepare SonarScanner') {
            steps {
                script {
                    // Geçici dizini temizle ve oluştur
                    sh 'rm -rf $TEMP_TOOLS && mkdir -p $TEMP_TOOLS'
                    // Dotnet SonarScanner'ı yükle
                    sh 'dotnet tool install --tool-path $TEMP_TOOLS dotnet-sonarscanner --version 10.3.0'
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

        stage('Docker Build') {
            steps {
                script {
                    // Jenkins container'ından Minikube Docker daemon'a bağlanmak için
                    sh 'eval $(minikube -p minikube docker-env) || true'
                    sh 'docker build -t testjenkins:latest .'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'kubectl apply -f TestJenkins/k8s/deployment.yaml'
                    sh 'kubectl apply -f TestJenkins/k8s/service.yaml'
                }
            }
        }
    }

    post {
        success { echo "Pipeline başarılı ✅" }
        failure { echo "Pipeline başarısız ❌" }
    }
}
