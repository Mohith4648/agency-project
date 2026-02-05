pipeline {
    agent any

    environment {
        // --- CONFIGURATION ---
        GIT_URL = "https://github.com/Mohith4648/agency-project.git"
        IMAGE_NAME = "agency-project"
        TAG = "v1"
        
        // --- UPDATED CREDENTIAL ID ---
        SONAR_TOKEN_ID = "sonarqube-token" 
        DOCKER_CRED_ID = "dockerentry"
        K8S_CRED_ID    = "k8s-kubeconfig"
        
        // --- SONARQUBE DETAILS ---
        SONAR_PROJECT_KEY = "Mohith4648_agency-project"
        SONAR_ORG_KEY     = "mohith4648"
        
        // --- KUBERNETES DETAILS ---
        K8S_DEPLOYMENT_NAME = "agency-proj-deployment"
    }

    stages {
        stage('1. Setup & Workspace Cleanup') {
            steps {
                cleanWs()
                // Pulling from the agency-project repo specifically
                git branch: 'main', url: "${env.GIT_URL}"
            }
        }

        stage('2. SonarQube Static Analysis') {
            steps {
                script {
                    // This grabs the tool we just configured in Step 1
                    def scannerHome = tool 'sonar-scanner'
                    
                    // withSonarQubeEnv allows Jenkins to use your sonarqube-token
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
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
                withCredentials([usernamePassword(credentialsId: "${env.DOCKER_CRED_ID}", 
                                                 passwordVariable: 'DOCKER_PASS', 
                                                 usernameVariable: 'DOCKER_USER')]) {
                    script {
                        echo "Building Docker Image..."
                        sh "docker build -t ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG} ."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                        sh "docker push ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG}"
                    }
                }
            }
        }

        stage('4. Kubernetes Production Deployment') {
            steps {
                withCredentials([file(credentialsId: "${env.K8S_CRED_ID}", variable: 'KUBECONFIG')]) {
                    script {
                        echo "Deploying to Kubernetes..."
                        
                        // Download kubectl locally in the workspace
                        sh 'curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                        sh 'chmod +x ./kubectl'
                        
                        // Apply deployment and service
                        sh "./kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml --validate=false"
                        sh "./kubectl --kubeconfig=${KUBECONFIG} rollout restart deployment/${env.K8S_DEPLOYMENT_NAME}"
                        
                        echo "SUCCESS: Agency Project is live!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution finished."
        }
    }
}
