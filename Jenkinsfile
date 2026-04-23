pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        IMAGE_NAME = 'daneshkabade45/demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    tools {
        maven "maven-3.8.4"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/danesh854/ecomm_products_api.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes (Backend Namespace)') {
            steps {
                sh '''
                set -e

                export KUBECONFIG=/var/lib/jenkins/.kube/config

                echo "Applying manifests to backend namespace..."
                kubectl apply -f Deployment.yml -n backend

                echo "Updating deployment image..."
                kubectl set image deployment/productsapideployment \
                products-container=$IMAGE_NAME:$IMAGE_TAG -n backend

                echo "Waiting for rollout..."
                kubectl rollout status deployment/productsapideployment -n backend
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful 🎉'
        }
        failure {
            echo '❌ Pipeline Failed'
        }
    }
}
