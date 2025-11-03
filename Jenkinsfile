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
                script {
                    // Use bat for Windows commands
                    bat """
                        echo "Current directory: %CD%"
                        dir
                    """
                    
                    // Run Docker commands with bat
                    bat """
                        docker run --rm -u root:root ^
                        -v "%CD%":/workspace ^
                        -w /workspace ^
                        ubuntu:22.04 ^
                        bash -c "
                            echo '=== Installing dependencies ===' && ^
                            apt-get update && ^
                            apt-get install -y git && ^
                            echo '=== Initializing submodules ===' && ^
                            git config --global --add safe.directory '*' && ^
                            git submodule update --init --recursive && ^
                            echo '=== Current Directory ===' && ^
                            pwd && ^
                            echo '=== Files in Workspace ===' && ^
                            ls -la && ^
                            echo '=== Validating CoinOS configuration ===' && ^
                            cat configs/coinos-base.yaml
                        "
                    """
                }
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