pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:latest
                command:
                - cat
                tty: true
                volumeMounts:
                - mountPath: /var/run/docker.sock
                  name: docker-sock
              - name: kubectl
                image: bitnami/kubectl:latest
                command:
                - cat
                tty: true
              volumes:
              - name: docker-sock
                hostPath:
                  path: /var/run/docker.sock
            '''
        }
    }
    
    environment {
        GIT_REPO = 'https://github.com/RafaelM1994/tii-hextris-devops-assessment.git'
        HEXTRIS_DIR = 'hextris'  // Folder inside the repo to navigate into
        DOCKER_IMAGE = 'hextris-app'
        K8S_DEPLOYMENT = 'hextris-app'
        K8S_NAMESPACE = 'default'
    }

    parameters {
        string(name: 'AWS_REGION', defaultValue: 'ap-east-1', description: 'AWS region where ECR and EKS are located')
    }

    stages {
       stage('Checkout Code') {
            steps {
                container('docker') {
                    script {
                        // Checkout the GitHub repository
                        git url: "${GIT_REPO}", branch: 'main'  // Use the correct branch if needed
                    }
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                container('docker') {
                    script {
                        def buildTag = "${env.BUILD_TAG}"
                        echo "Using Build Tag: ${buildTag}"
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${buildTag} ./hextris
                            docker tag ${DOCKER_IMAGE}:${buildTag} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                        def AWS_ACCOUNT = sh(script: "aws sts get-caller-identity --query 'Account' --output text", returnStdout: true).trim()
                        def REGISTRY = "${AWS_ACCOUNT}.dkr.ecr.${params.AWS_REGION}.amazonaws.com"                        
                        sh """
                            aws ecr get-login-password --region ${params.AWS_REGION} | docker login --username AWS --password-stdin ${params.REGISTRY}
                            docker push ${params.IMAGE_URI}:${BUILD_TAG}    
                        """
                    }
                }
            }
        }
        stage('Update and Apply Kubernetes Deployment') {
            steps {
                container('kubectl') {
                    script {
                        sh """
                            kubectl set image deployment/${K8S_DEPLOYMENT} ${K8S_DEPLOYMENT}=${DOCKER_IMAGE}:${BUILD_TAG} -n ${K8S_NAMESPACE}
                            kubectl rollout restart deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace after build
        }
    }
}