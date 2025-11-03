pipeline {
    agent any
    
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Andy11011/CoinOS.git'
            }
        }
        
        stage('Build') {
            steps {
                bat 'echo "Current directory: %CD%"'
                bat 'dir'
                                
                // Step 1: Start container and keep it running
                bat 'docker run -d --name coinos-build -u root:root -v "%CD%":/workspace -w /workspace ubuntu:22.04 tail -f /dev/null'

                // Step 2: Configure git safe directory
                bat 'docker exec coinos-build bash -c "apt-get update && apt-get install -y git"'
                
                // Step 3: Show directory and files
                bat 'docker exec coinos-build bash -c "pwd && ls -la"'

                // Step 4: Display config file
                bat 'docker exec coinos-build bash -c "cat configs/coinos-base.yaml"'
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