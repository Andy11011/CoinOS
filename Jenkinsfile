pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            args '-u root:root'
        }
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out CoinOS repository...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing build dependencies...'
                sh '''
                    apt-get update
                    apt-get install -y git
                '''
            }
        }
        
        stage('Validate Config') {
            steps {
                echo 'Validating CoinOS configuration...'
                sh 'cat configs/coinos-base.yaml'
            }
        }
        
        stage('Build Image') {
            steps {
                echo 'Building CoinOS image...'
                sh 'echo "Build command will go here (needs more setup)"'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                // archiveArtifacts artifacts: 'output/*.img', fingerprint: true
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