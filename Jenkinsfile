pipeline {
    agent any

    environment {
        EC2_HOST = "100.31.157.122"   // Replace with your EC2 public IP
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone from your public GitHub repo using HTTPS
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/master']],   // ✅ using master branch
                          userRemoteConfigs: [[url: 'https://github.com/rohan4linux/hello-world-new.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                // Use Maven tool configured in Jenkins (Global Tool Configuration)
                withMaven(maven: 'maven-3.9.12') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Inject AWS key from Jenkins Credentials Manager
                withCredentials([sshUserPrivateKey(credentialsId: 'aws-ec2-key', keyFileVariable: 'AWS_KEY')]) {
                    sh '''
                    # Clean old deployment (WAR + exploded dir)
                    ssh -i $AWS_KEY -o StrictHostKeyChecking=no ubuntu@$EC2_HOST \
                        "sudo rm -rf /var/lib/tomcat10/webapps/webapp /var/lib/tomcat10/webapps/webapp.war"

                    # Copy new WAR (correct path inside webapp/target)
                    scp -i $AWS_KEY -o StrictHostKeyChecking=no webapp/target/*.war ubuntu@$EC2_HOST:/tmp/

                    # Move WAR into Tomcat webapps and restart Tomcat
                    ssh -i $AWS_KEY -o StrictHostKeyChecking=no ubuntu@$EC2_HOST \
                        "sudo mv /tmp/*.war /var/lib/tomcat10/webapps/ && sudo systemctl restart tomcat10"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Access your app at http://$EC2_HOST:8080/webapp"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins logs."
        }
    }
}

