pipeline {
    agent any

    stages {
        stage('Build Blue Image') {
            steps {
                sh 'docker build -t vle8-blue:1.0 ./blue'
            }
        }

        stage('Build Green Image') {
            steps {
                sh 'docker build -t vle8-green:2.0 ./green'
            }
        }

        stage('Load Images to Minikube') {
            steps {
                sh 'sudo -u ubuntu /usr/local/bin/minikube image load vle8-blue:1.0'
                sh 'sudo -u ubuntu /usr/local/bin/minikube image load vle8-green:2.0'
            }
        }

        stage('Deploy Blue and Green') {
            steps {
                sh 'kubectl apply -f k8s/blue-deployment.yaml'
                sh 'kubectl apply -f k8s/green-deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify Deployments') {
            steps {
                sh 'kubectl rollout status deployment/blue-deployment --timeout=120s'
                sh 'kubectl rollout status deployment/green-deployment --timeout=120s'
                sh 'kubectl get pods'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''kubectl patch service vle8-service -p '{"spec":{"selector":{"app":"vle8","color":"green"}}}' '''
            }
        }

        stage('Rollback to Blue') {
            steps {
                sh '''kubectl patch service vle8-service -p '{"spec":{"selector":{"app":"vle8","color":"blue"}}}' '''
            }
        }
    }
}
