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
                
                // Single-line Docker command
                bat 'docker run --rm -u root:root -v "%CD%":/workspace -w /workspace ubuntu:22.04 bash -c "apt-get update && apt-get install -y git && git config --global --add safe.directory \"*\" && git submodule update --init --recursive && pwd && ls -la && cat configs/coinos-base.yaml"'
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