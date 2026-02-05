pipeline {
    agent any

    environment {
        // --- CONFIGURATION ---
        GIT_URL = "https://github.com/Mohith4648/agency-project.git"
        IMAGE_NAME = "agency-project"
        TAG = "v1"
        
        // --- UPDATED CREDENTIAL IDs ---
        SONAR_TOKEN_ID = "sonarqube-token" 
        DOCKER_CRED_ID = "docker-id" // This matches the ID you just created
        K8S_CRED_ID    = "k8s-kubeconfig"
        
        // --- SONARQUBE DETAILS ---
        SONAR_PROJECT_KEY = "Mohith4648_agency-project"
        SONAR_ORG_KEY     = "mohith4648"
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
                            -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                            -Dsonar.organization=${env.SONAR_ORG_KEY} \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=https://sonarcloud.io"
                    }
                }
            }
        }

        stage('3. Build & Push Image') {
            steps {
                // Using the new 'docker-id' to grab your username and token
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CRED_ID}", 
                                                 passwordVariable: 'DOCKER_PASS', 
                                                 usernameVariable: 'DOCKER_USER')]) {
                    script {
                        echo "Building image for user: ${DOCKER_USER}"
                        sh 'docker build -t $DOCKER_USER/$IMAGE_NAME:$TAG .'
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $DOCKER_USER/$IMAGE_NAME:$TAG'
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
