pipeline {
    agent any

    environment {
        SSH_CREDENTIALS = 'ec2-ssh-key'  // Jenkins credential ID for SSH key
        EC2_USER = 'ubuntu'              // EC2 instance user
        EC2_HOST = '35.173.236.209'     // Staging EC2 public IP
        APP_DIR = '/home/ubuntu/app'    // Deployment directory on EC2
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/v-2212221/simple-java-maven-app.git'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    if ! mvn -version > /dev/null 2>&1; then
                        echo "Maven not found, installing..."
                        sudo apt-get update
                        sudo apt-get install -y maven
                    else
                        echo "Maven is already installed"
                    fi
                    mvn clean package
                '''
            }
        }

        stage('Check JAR') {
            steps {
                sh 'ls target/*.jar'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([SSH_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'mkdir -p ${APP_DIR}'
                        scp -o StrictHostKeyChecking=no target/simple-java-maven-app.jar ${EC2_USER}@${EC2_HOST}:${APP_DIR}/
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'pkill -f simple-java-maven-app.jar || true'
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'nohup java -jar ${APP_DIR}/simple-java-maven-app.jar > app.log 2>&1 &'
                    """
                }
            }
        }
    }
}
