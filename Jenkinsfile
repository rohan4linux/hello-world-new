@Library('jenkins-shared-lib') _

pipeline {
    agent any

    environment {
        EC2_HOST = "100.31.157.122"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/master']],
                          userRemoteConfigs: [[url: 'https://github.com/rohan4linux/hello-world-new.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                withMaven(maven: 'maven-3.9.12') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Deploy') {
            steps {
                // Call the shared library function
                deployToEC2('webapp/target/*.war', EC2_HOST, 'aws-ec2-key')
            }
        }
    }

    post {
        success {
            echo "✅ Deployment is successful! Access your app at http://${EC2_HOST}:8080/webapp"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}

