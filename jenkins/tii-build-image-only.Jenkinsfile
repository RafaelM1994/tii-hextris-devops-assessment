pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: podman
                image: quay.io/podman/stable:latest
                command:
                - cat
                tty: true
                securityContext:
                  runAsUser: 0
                  privileged: true
              volumes:
              - name: podman-socket
                emptyDir: {}
            '''
        }
    }

    environment {
        GIT_REPO = 'https://github.com/RafaelM1994/tii-hextris-devops-assessment.git'
        PODMAN_IMAGE = 'hextris-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                container('podman') {
                    script {
                        git url: "${GIT_REPO}", branch: 'main'
                    }
                }
            }
        }

        stage('Build and Tag Image') {
            steps {
                container('podman') {
                    script {
                        def buildTag = "${env.BUILD_TAG}"
                        echo "Using Build Tag: ${buildTag}"
                        sh """
                            podman build -t ${PODMAN_IMAGE}:${buildTag} ./hextris
                            podman tag ${PODMAN_IMAGE}:${buildTag} ${PODMAN_IMAGE}:latest
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