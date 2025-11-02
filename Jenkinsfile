pipeline {
    agent none
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'ubuntu:22.04'
                    args "-u root:root -v /${WORKSPACE.replaceAll('\\\\', '/')}:/workspace"
                    workingDir '/workspace'
                }
            }
            steps {
                echo 'Checking out CoinOS repository...'
                checkout scm
                
                echo 'Installing dependencies...'
                sh '''
                    apt-get update
                    apt-get install -y git
                    pwd
                    ls -la
                '''
                
                echo 'Initializing rpi-image-gen submodule...'
                sh '''
                    git config --global --add safe.directory '*'
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