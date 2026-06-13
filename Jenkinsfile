pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev',
                url: 'https://github.com/YOUR-REPO.git'
            }
        }

        stage('SonarQube Test') {
            steps {
                echo 'SonarQube integration will go here'
            }
        }
    }
}
