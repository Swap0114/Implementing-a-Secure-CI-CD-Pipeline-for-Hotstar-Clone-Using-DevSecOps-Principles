pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "swap0114/hotstar-clone"
        SONAR_PROJECT = "hotstar-clone"
    }
    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=hotstar-clone \
                        -Dsonar.projectName=hotstar-clone \
                        -Dsonar.sources=.
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }
        stage('Docker Scout Scan') {
            steps {
                sh "docker scout cves ${DOCKER_IMAGE}:latest || true"
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s/"
            }
        }
        stage('OWASP ZAP Scan') {
            steps {
                sh '''
                    docker run --rm \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py \
                    -t http://$(kubectl get svc hotstar-clone \
                    -o jsonpath="{.status.loadBalancer.ingress[0].hostname}") \
                    -r zap-report.html || true
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
