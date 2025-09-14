pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token') // Jenkins'deki token ID
        DOTNET_CLI_HOME = '/var/jenkins_home'
        PATH = "/usr/share/dotnet:/root/.dotnet/tools:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
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
                    def projectDir = "${env.WORKSPACE}/TestJenkins"

                    sh """
                        # PATH ve DOTNET_CLI_HOME ayarları
                        export DOTNET_CLI_HOME=$DOTNET_CLI_HOME
                        export PATH=/usr/share/dotnet:/root/.dotnet/tools:\$PATH

                        cd ${projectDir}

                        echo "Starting SonarScanner..."
                        /root/.dotnet/tools/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=${SONARQUBE} /d:sonar.host.url=http://sonarqube:9000

                        echo "Building project..."
                        dotnet build TestJenkins.csproj -c Release

                        echo "Running tests..."
                        dotnet test TestJenkins.csproj -c Release

                        echo "Ending SonarScanner..."
                        /root/.dotnet/tools/dotnet-sonarscanner end /d:sonar.login=${SONARQUBE}
                    """
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                echo 'Skipping Docker stage for now due to earlier failures'
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed ❌"
        }
        success {
            echo "Pipeline succeeded ✅"
        }
    }
}
