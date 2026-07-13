pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        IMAGE_NAME = "dineshd1575/springboot-app:latest"
        AWS_REGION = "eu-north-1"
        EKS_CLUSTER = "devops-cluster"
        HOME = "/var/lib/jenkins"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/dineshd1575/jenkins-devops-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                        docker logout
                    '''
                }
            }
        }

        stage('Debug Environment') {
            steps {
                sh '''
                    echo "==============================="
                    echo "Current User"
                    whoami

                    echo "==============================="
                    echo "Current Directory"
                    pwd

                    echo "==============================="
                    echo "HOME"
                    echo $HOME

                    echo "==============================="
                    echo "KUBECONFIG"
                    echo $KUBECONFIG

                    echo "==============================="
                    echo "AWS Identity"
                    aws sts get-caller-identity

                    echo "==============================="
                    echo "AWS Version"
                    aws --version

                    echo "==============================="
                    echo "Kubectl Version"
                    kubectl version --client

                    echo "==============================="
                    echo "Current Context"
                    kubectl config current-context

                    echo "==============================="
                    echo "Cluster Info"
                    kubectl cluster-info

                    echo "==============================="
                    echo "Nodes"
                    kubectl get nodes

                    echo "==============================="
                    echo "Workspace Files"
                    ls -l

                    echo "==============================="
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    export HOME=/var/lib/jenkins
                    export KUBECONFIG=/var/lib/jenkins/.kube/config

                    aws eks update-kubeconfig \
                      --region $AWS_REGION \
                      --name $EKS_CLUSTER

                    kubectl apply -f deployment.yaml --validate=false
                    kubectl apply -f service.yaml --validate=false

                    kubectl get deployments
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            cleanWs()
        }
    }
}
