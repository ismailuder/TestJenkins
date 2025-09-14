pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')  // Jenkins içinde oluşturduğun token
        IMAGE_NAME = "testjenkins:latest"
        KUBE_CONFIG = "/home/jenkins/.kube/config"
        DOTNET_CLI_HOME = "/var/jenkins_home"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                script {
                    // Workspace yolunu belirle
                    def appPath = "${env.WORKSPACE}/src/TestJenkins"
                    sh """
                        export PATH=/usr/share/dotnet:/root/.dotnet/tools:$PATH
                        export DOTNET_CLI_HOME=/var/jenkins_home

                        cd $appPath

                        # SonarScanner begin
                        dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000

                        # Build ve test
                        dotnet build TestJenkins.csproj -c Release
                        dotnet test TestJenkins.csproj -c Release

                        # SonarScanner end
                        dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                    """
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh """
                    docker build -t $IMAGE_NAME $WORKSPACE
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
