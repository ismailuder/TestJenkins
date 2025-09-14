pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')  // Jenkins içinde oluşturduğun token
        IMAGE_NAME = "testjenkins:latest"
        KUBE_CONFIG = "/home/jenkins/.kube/config"
        DOTNET_CLI_HOME = "/var/jenkins_home"  // Sonar için gerekli
        PATH = "/usr/share/dotnet:/root/.dotnet/tools:$PATH"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Check Files') {
            steps {
                sh 'ls -R $WORKSPACE'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                script {
                    // Repo içindeki proje path'ini kontrol et
                    def projectDir = "${WORKSPACE}/src/TestJenkins" // Eğer csproj başka yerdeyse burayı değiştir

                    sh """
                        cd ${projectDir}
                        dotnet sonarscanner begin /k:'TestJenkins' /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins.csproj -c Release
                        dotnet test TestJenkins.csproj -c Release
                        dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                    """
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh """
                    docker build -t $IMAGE_NAME .
                    minikube image load $IMAGE_NAME
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! 🎉"
        }
        failure {
            echo "Pipeline failed. ❌"
        }
    }
}
