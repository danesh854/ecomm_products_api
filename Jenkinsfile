pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        IMAGE_NAME = 'daneshkabade45/demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        NAMESPACE = 'backend'
        DEPLOYMENT_NAME = 'productsapideployment'
        CONTAINER_NAME = 'products-container'
    }

    tools {
        maven "maven-3.8.4"
    }

    stages {

        // ❌ No Clone stage (Jenkins already checks out code)

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

                echo "📦 Applying manifests..."
                kubectl apply -f Deployment.yml -n $NAMESPACE

                echo "🚀 Updating deployment image..."
                kubectl set image deployment/$DEPLOYMENT_NAME \
                $CONTAINER_NAME=$IMAGE_NAME:$IMAGE_TAG -n $NAMESPACE

                echo "⏳ Waiting for rollout..."
                kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Backend Deployment Successful 🎉'
        }
        failure {
            echo '❌ Pipeline Failed'
        }
    }
}
