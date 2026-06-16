pipeline {
    agent any

   

    environment {
        S3_BUCKET = 'r-assign-1-508563139089-us-east-1-an'
        APACHE_SERVER = '54.196.230.222'
        ARTIFACT_NAME = "website-${BUILD_NUMBER}.zip"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'dev',
                    url: 'https://github.com/rashmipathinayaka/sysco-1.git'
            }
        }


        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    sonar-scanner \
                    -Dsonar.projectKey=assignment-1 \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Create Artifact') {
            steps {
                sh '''
                zip -r ${ARTIFACT_NAME} .
                '''
            }
        }

        stage('Upload To S3') {
            steps {
                withAWS(credentials: 'AWS', region: 'us-east-1') {
                    sh '''
                    aws s3 cp ${ARTIFACT_NAME} s3://${S3_BUCKET}/${ARTIFACT_NAME}
                    '''
                }
            }
        }

        stage('Deploy To Apache') {
            steps {
                sshagent(credentials: ['server-2(apache)']) {

                    sh """
                    scp -o StrictHostKeyChecking=no \
                    ${ARTIFACT_NAME} \
                    ec2-user@${APACHE_SERVER}:/tmp/
                    """

                    sh """
                    ssh -o StrictHostKeyChecking=no \
                    ec2-user@${APACHE_SERVER} '

                    sudo rm -rf /var/www/html/*

                    sudo unzip -o /tmp/${ARTIFACT_NAME} \
                    -d /var/www/html/

                    sudo systemctl restart httpd
                    '
                    """
                }
            }
        }

        stage('Validate Deployment') {
            steps {
                script {

                    def status = sh(
                        script: """
                        curl -s -o /dev/null \
                        -w "%{http_code}" \
                        http://${APACHE_SERVER}
                        """,
                        returnStdout: true
                    ).trim()

                    echo "HTTP Status Code: ${status}"

                    if (status != "200") {
                        error("Deployment validation failed")
                    }
                }
            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}