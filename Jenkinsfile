pipeline {
    agent any

    environment {
        // --- CONFIGURATION ---
        GIT_URL = "https://github.com/Mohith4648/agency-project.git"
        IMAGE_NAME = "agency-project"
        TAG = "v1"
        
        // --- UPDATED CREDENTIAL IDs ---
        SONAR_TOKEN_ID = "sonarqube-token" 
        DOCKER_CRED_ID = "mohith4648" // Updated to match your verified ID
        K8S_CRED_ID    = "k8s-kubeconfig"
        
        // --- SONARQUBE DETAILS ---
        SONAR_PROJECT_KEY = "Mohith4648_agency-project"
        SONAR_ORG_KEY     = "mohith4648"
    }

    stages {
        stage('1. Setup & Workspace Cleanup') {
            steps {
                cleanWs()
                git branch: 'main', url: "${env.GIT_URL}"
            }
        }

        stage('2. SonarQube Static Analysis') {
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
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CRED_ID}", 
                                                 passwordVariable: 'DOCKER_PASS', 
                                                 usernameVariable: 'DOCKER_USER')]) {
                    script {
                        echo "Building and Pushing image for: ${DOCKER_USER}"
                        // Using single quotes for safety
                        sh 'docker build -t $DOCKER_USER/$IMAGE_NAME:$TAG .'
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $DOCKER_USER/$IMAGE_NAME:$TAG'
                    }
                }
            }
        }

        stage('4. Kubernetes Deployment') {
            steps {
                withCredentials([file(credentialsId: "${env.K8S_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh "kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml"
                }
            }
        }
    }
}
