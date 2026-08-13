pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
    	stage('Docker Build') {
            steps {
                sh 'docker build -t hello-world:1.0 .'
            }
        }

    }
}

