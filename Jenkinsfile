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
                    // Proje dizini
                    def projectDir = "${env.WORKSPACE}/TestJenkins"

                    // SonarScanner begin
                    sh """
                        export PATH=$PATH
                        export DOTNET_CLI_HOME=$DOTNET_CLI_HOME
                        cd ${projectDir}
                        dotnet sonarscanner begin /k:TestJenkins /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000
                    """

                    // Build
                    sh "dotnet build ${projectDir}/TestJenkins.csproj -c Release"

                    // Test
                    sh "dotnet test ${projectDir}/TestJenkins.csproj -c Release"

                    // SonarScanner end
                    sh "dotnet sonarscanner end /d:sonar.login=$SONARQUBE"
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
