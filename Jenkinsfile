pipeline {
    agent any

    environment {
        TEMP_TOOLS = "${WORKSPACE}/.temp_dotnet_tools"
        PATH = "${TEMP_TOOLS}:${env.PATH}"
        BUILD_CONFIGURATION = "Release"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Prepare .NET SonarScanner') {
            steps {
                script {
                    sh 'rm -rf $TEMP_TOOLS && mkdir -p $TEMP_TOOLS'
                    sh 'dotnet tool install --tool-path $TEMP_TOOLS dotnet-sonarscanner --version 10.3.0'
                }
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        $TEMP_TOOLS/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins/TestJenkins.csproj -c $BUILD_CONFIGURATION
                        dotnet test TestJenkins/TestJenkins.csproj -c $BUILD_CONFIGURATION
                        $TEMP_TOOLS/dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${env.WORKSPACE}") {
                    // Eğer Dockerfile TestJenkins altındaysa:
                    sh 'docker build -f TestJenkins/Dockerfile -t testjenkins:latest .'
                }
            }
        }

        stage('Deploy to Local Kubernetes') {
			steps {
				dir("${env.WORKSPACE}/TestJenkins") {
					sh 'kubectl apply -f k8s/testjenkins-deployment.yaml'
				}
			}
		}
    }

    post {
        success { echo "Pipeline başarılı ✅" }
        failure { echo "Pipeline başarısız ❌" }
    }
}
