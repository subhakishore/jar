pipeline {
    agent any

    environment {
        EC2_HOST = 'ec2-3-1-23-37.ap-southeast-1.compute.amazonaws.com'
        EC2_USER = 'ubuntu'
        APP_DIR = '/home/ubuntu/app'
        SSH_KEY_ID = 'ec2-ssh'
    }

    tools {
        maven 'Maven3'
        jdk 'jdk17'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/subhakishore/jar.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: [env.SSH_KEY_ID]) {
                    sh '''
                        JAR_FILE=$(ls target/*.jar | head -n 1)
                        echo "Deploying $JAR_FILE to EC2..."
                        scp -o StrictHostKeyChecking=no $JAR_FILE ${EC2_USER}@${EC2_HOST}:${APP_DIR}/app.jar
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "pkill -f 'java -jar' || true"
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "nohup java -jar ${APP_DIR}/app.jar > ${APP_DIR}/app.log 2>&1 &"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
