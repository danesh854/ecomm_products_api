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
                git branch: 'main', url: 'https://github.com/danesh854/ecomm_products_api.git'
            }
        }

        stage('Build JAR') {
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

        stage('Deploy to EKS') {
            steps {
                sh '''
                set -e

                export KUBECONFIG=/var/lib/jenkins/.kube/config
                export AWS_DEFAULT_REGION=ap-southeast-1

                echo "🔍 Checking AWS identity..."
                aws sts get-caller-identity

                echo "🔄 Updating kubeconfig..."
                aws eks update-kubeconfig --region ap-southeast-1 --name my-cluster

                echo "📦 Applying Kubernetes manifests..."
                kubectl apply -f Deployment.yml

                echo "🚀 Updating deployment image..."
                kubectl set image deployment/products-deployment \
                products-container=$IMAGE_NAME:$IMAGE_TAG

                echo "⏳ Waiting for rollout..."
                kubectl rollout status deployment/products-deployment
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
