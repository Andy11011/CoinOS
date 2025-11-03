pipeline {
    agent any // ← This creates a workspace
    
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
                    args '-u root:root'
                    reuseNode true
                }
            }
            steps {
                echo "Workspace path: ${WORKSPACE}"
                sh 'pwd && ls -la'
                // Rest of your steps...
            }
        }
    }
}