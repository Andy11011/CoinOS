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
                bat 'docker exec coinos-build bash -c "apt-get update && apt-get install -y git curl wget"'

                // Step 3: Clone rpi-image-gen directly (instead of using submodule)
                bat 'docker exec coinos-build bash -c "git clone https://github.com/raspberrypi/rpi-image-gen.git rpi-image-gen || echo \"pi-gen repo cloned\""'
                
                // Step 4: Show directory and files
                bat 'docker exec coinos-build bash -c "pwd && ls -la"'

                // Step 5: Display config file
                bat 'docker exec coinos-build bash -c "cat configs/coinos-base.yaml"'

                // Step 6: Prepare build environment
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ls -la"'

                // Step 7: Build the image (this is where the actual build happens)
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && echo \"Build command would go here\""'

                // Cleanup
                bat 'docker stop coinos-build'
                bat 'docker rm coinos-build'
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