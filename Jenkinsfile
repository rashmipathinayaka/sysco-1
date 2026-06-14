pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev',
                url: 'https://github.com/rashmipathinayaka/sysco-1'
            }
        }

        stage('SonarQube Test') {
            steps {
                echo 'SonarQube integration will go here'
            }
        }
    }
}
