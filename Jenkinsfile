pipeline {
    agent any

    environment {
        GIT_URL = "https://github.com/Mohith4648/agency-project.git"
        IMAGE_NAME = "agency-project"
        TAG = "v1"
        SONAR_TOKEN_ID = "sonarqube-token" 
        DOCKER_CRED_ID = "mohith4648" // This ID connects to your Token safely
        K8S_CRED_ID    = "k8s-kubeconfig"
    }

    stages {
        stage('1. Setup') {
            steps {
                cleanWs()
                git branch: 'main', url: "${env.GIT_URL}"
            }
        }

        stage('2. SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Mohith4648_agency-project \
                            -Dsonar.organization=mohith4648 \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=https://sonarcloud.io"
                    }
                }
            }
        }

        stage('3. Build & Push Image') {
            steps {
                // This block "hides" your token while it works
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CRED_ID}", 
                                                 passwordVariable: 'DOCKER_PASS', 
                                                 usernameVariable: 'DOCKER_USER')]) {
                    script {
                        echo "Logging into Docker Hub as: ${DOCKER_USER}"
                        sh "docker build -t ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG} ."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                        sh "docker push ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG}"
                    }
                }
            }
        }

        stage('4. Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${env.K8S_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml"
                }
            }
        }
    }
}
