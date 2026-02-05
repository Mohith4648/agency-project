pipeline {
    agent any

    environment {
        // --- SONARQUBE CONFIGURATION ---
        SONAR_PROJECT_KEY = "Mohith4648_internfile"
        SONAR_ORG_KEY     = "mohith4648"
        
        // --- DOCKER CONFIGURATION ---
        IMAGE_NAME = "intern-project"
        TAG = "v1"
        CRED_ID = "dockerentry" 
        
        // --- KUBERNETES CONFIGURATION ---
        K8S_DEPLOYMENT_NAME = "intern-proj-deployment"
        KUBECONFIG_PATH = "scripts/myconfig.yaml" 
    }

    stages {
        stage('1. Setup & Workspace Cleanup') {
            steps {
                cleanWs()
                git branch: 'main', url: 'https://github.com/Mohith4648/internfile.git'
            }
        }

        stage('2. SonarQube Static Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    script {
                        echo "Starting Code Analysis on SonarCloud..."
                        sh """
                            docker run --rm \
                            -v ${WORKSPACE}:/usr/src \
                            -e SONAR_TOKEN=${SONAR_TOKEN} \
                            -e SONAR_HOST_URL="https://sonarcloud.io" \
                            sonarsource/sonar-scanner-cli \
                            -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                            -Dsonar.organization=${env.SONAR_ORG_KEY} \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('3. Build & Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${env.CRED_ID}", 
                                                 passwordVariable: 'DOCKER_PASS', 
                                                 usernameVariable: 'DOCKER_USER')]) {
                    script {
                        echo "Building and Pushing Image to Docker Hub..."
                        sh "docker build -t ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG} ."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                        sh "docker push ${DOCKER_USER}/${env.IMAGE_NAME}:${env.TAG}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('4. Kubernetes Production Deployment') {
            steps {
                script {
                    echo "Checking for kubectl and deploying..."
                    
                    if (fileExists(env.KUBECONFIG_PATH)) {
                        // Download portable kubectl
                        sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                        sh 'chmod +x ./kubectl'
                        
                        // Execute Deployment using the downloaded binary
                        sh "./kubectl --kubeconfig=${env.KUBECONFIG_PATH} apply -f deployment.yaml"
                        sh "./kubectl --kubeconfig=${env.KUBECONFIG_PATH} rollout restart deployment/${env.K8S_DEPLOYMENT_NAME}"
                        sh "./kubectl --kubeconfig=${env.KUBECONFIG_PATH} rollout status deployment/${env.K8S_DEPLOYMENT_NAME}"
                        
                        echo "------------------------------------------------------------"
                        echo "SUCCESS: Self-Healing Kubernetes Deployment Complete!"
                        echo "------------------------------------------------------------"
                    } else {
                        error "ERROR: ${env.KUBECONFIG_PATH} not found in repository!"
                    }
                }
            }
        }
    } // End of Stages

    post {
        always {
            echo "Pipeline execution finished."
        }
    }
} // End of Pipeline
