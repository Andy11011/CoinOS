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
                                
                // Step 1: Register QEMU on Docker host (this container will exit after registering)
                bat 'docker run --rm --privileged multiarch/qemu-user-static --reset -p yes || echo "QEMU registration attempted"'
                
                // Step 2: Start build container
                bat 'docker run -d --name coinos-build --privileged -u root:root -v "%CD%":/workspace -w /workspace debian:bookworm tail -f /dev/null'

                // Step 3: Update and install basic dependencies
                bat 'docker exec coinos-build bash -c "apt-get update && apt-get install -y git curl wget"'

                // Step 4: Clone rpi-image-gen directly (instead of using submodule)
                bat 'docker exec coinos-build bash -c "git clone https://github.com/raspberrypi/rpi-image-gen.git rpi-image-gen || echo \"pi-gen repo cloned\""'
                
                // Step 5: Show directory and files
                bat 'docker exec coinos-build bash -c "pwd && ls -la"'

                // Step 6: Display config file
                bat 'docker exec coinos-build bash -c "cat configs/coinos-base.yaml"'

                // Step 7: Install rpi-image-gen dependencies
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ./install_deps.sh"'

                // Step 8: Install QEMU user static for ARM emulation
                bat 'docker exec coinos-build bash -c "apt-get install -y qemu-user-static binfmt-support"'

                // Step 9: Verify binfmt is working
                bat 'docker exec coinos-build bash -c "ls -la /proc/sys/fs/binfmt_misc/ || echo \\"binfmt_misc check\\""'

                // Step 10: Build using our existing coinos-base.yaml config
                bat 'docker exec coinos-build bash -c "cd rpi-image-gen && ./rpi-image-gen build -c ../configs/coinos-base.yaml"'

                // Step 11: Copy built image to workspace (so it survives container cleanup)
                bat 'docker exec coinos-build bash -c "cp -r rpi-image-gen/image/ . || echo No image directory found"'

                // Step 12: Check if image was created
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