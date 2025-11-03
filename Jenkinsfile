pipeline {
    agent any
    
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Andy11011/CoinOS.git'
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'ubuntu:22.04'
                    args '-u root:root -w /workspace'
                    reuseNode true
                }
            }
            steps {
                echo "Workspace path: ${WORKSPACE}"
                sh 'pwd && ls -la'
                
                echo 'Installing dependencies...'
                sh '''
                    apt-get update
                    apt-get install -y git
                '''
                
                echo 'Initializing rpi-image-gen submodule...'
                sh '''
                    git config --global --add safe.directory "*"
                    git submodule update --init --recursive
                '''
                
                echo 'Validating CoinOS configuration...'
                sh 'cat configs/coinos-base.yaml'
                
                echo 'Building CoinOS image...'
                sh 'echo "Build command will go here"'
            }
        }
    }
    
    post {
        success {
            echo 'CoinOS build completed successfully!'
        }
        failure {
            echo 'CoinOS build failed!'
        }
    }
}