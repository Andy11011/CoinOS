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
                                
                // Step 1: Start container with Debian 12 (bookworm)
                bat 'docker run -d --name coinos-build -u root:root -v "%CD%":/workspace -w /workspace debian:bookworm tail -f /dev/null'

                // Step 2: Update and install basic dependencies
                bat 'docker exec coinos-build bash -c "apt-get update && apt-get install -y git curl wget"'

                // Step 3: Clone rpi-image-gen directly (instead of using submodule)
                bat 'docker exec coinos-build bash -c "git clone https://github.com/raspberrypi/rpi-image-gen.git rpi-image-gen || echo \"pi-gen repo cloned\""'
                
                // Step 4: Show directory and files
                bat 'docker exec coinos-build bash -c "pwd && ls -la"'

                // Step 5: Display config file
                bat 'docker exec coinos-build bash -c "cat configs/coinos-base.yaml"'

                // Step 6: Install rpi-image-gen dependencies
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ./install_deps.sh"'

                // Step 7: Prepare build environment
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ls -la"'

                // Step 8: Build using our existing coinos-base.yaml config
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ./rpi-image-gen build -c ../configs/coinos-base.yaml"'

                // Step 9: Copy built image to workspace (so it survives container cleanup)
                bat 'docker exec coinos-build bash -c "cp -r rpi-image-gen/image/ . || echo No image directory found"'

                // Step 10: Check if image was created
                bat 'docker exec coinos-build bash -c "ls -la image/ || echo No image directory"'
        
                // Cleanup
                bat 'docker stop coinos-build'
                bat 'docker rm coinos-build'
            }   
        }

       stage('Archive Artifacts') {
            steps {
                // Archive the built image
                archiveArtifacts artifacts: 'image/**/*.img', fingerprint: true
                
                // Also archive logs and configs
                archiveArtifacts artifacts: 'rpi-image-gen/build.log', fingerprint: true
                archiveArtifacts artifacts: 'configs/*.yaml', fingerprint: true
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
        always {
            // Clean up any remaining containers
            bat 'docker stop coinos-build || echo "No container to stop"'
            bat 'docker rm coinos-build || echo "No container to remove"'
        }
    }
}