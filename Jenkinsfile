pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

       stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343:sonar'
                }
            }
        }
 

        stage('Docker Build') {
            steps {
                sh 'docker build -t hello-world:1.0 .'
            }
        }

    }
}
