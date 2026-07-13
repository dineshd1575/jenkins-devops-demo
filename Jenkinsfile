pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('Sonar') {
            withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                sh '''
                mvn sonar:sonar \
                -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
    }


        stage('Docker') {
            steps {
                sh "docker build -t demo:${BUILD_NUMBER} ."
            }
        }

        stage('Deploy') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
}
