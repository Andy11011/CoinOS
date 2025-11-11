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
                
                // Step 1: Register QEMU on Docker host (this makes ARM emulation available)
                //bat 'docker run --rm --privileged multiarch/qemu-user-static --reset -p yes'

                // Step 2: Start build container with binfmt_misc mounted
                bat 'docker run -d --name coinos-build --privileged -u 1000:1000 -v "%CD%":/workspace -w /workspace debian:bookworm tail -f /dev/null'

                // Step 2.5: Install QEMU inside the container
                bat 'docker exec -u root coinos-build bash -c "apt-get update && apt-get install -y qemu-user-static binfmt-support"'

                // Step 2.6: Manually register QEMU interpreters (since the service didn't start)
                bat 'docker exec -u root coinos-build bash -c "update-binfmts --enable"'

                // Step 2.7: Verify it worked
                bat 'docker exec coinos-build bash -c "ls /proc/sys/fs/binfmt_misc/qemu-* && echo \"✓ QEMU registered\" || echo \"✗ QEMU not registered\""'

                // Check if binfmt_misc is available and see registered interpreters
                bat 'docker exec coinos-build bash -c "ls -la /proc/sys/fs/binfmt_misc/"'

                // See the QEMU ARM64 registration details
                bat 'docker exec coinos-build bash -c "cat /proc/sys/fs/binfmt_misc/qemu-aarch64"'

                // Step 3: Update and install basic dependencies
                bat 'docker exec -u root coinos-build bash -c "apt-get update && apt-get install -y git curl wget"'

                // Step 4: Clone rpi-image-gen directly (instead of using submodule)
                bat 'docker exec coinos-build bash -c "git clone https://github.com/raspberrypi/rpi-image-gen.git rpi-image-gen || echo \"pi-gen repo cloned\""'
                
                // Step 5: Show directory and files
                bat 'docker exec coinos-build bash -c "pwd && ls -la"'

                // Step 6: Display config files
                bat '''
                docker exec coinos-build bash -c "echo ===== config/coinos-config.ini ===== && cat /workspace/config/coinos-config.yaml || echo File not found"
                docker exec coinos-build bash -c "echo ===== layers/coinos-base.yaml ===== && cat /workspace/layer/coinos-base.yaml || echo File not found"
                '''

                // Step 7: Install rpi-image-gen dependencies
                bat 'docker exec -u root coinos-build bash -c "cd rpi-image-gen && ./install_deps.sh"'

                // Step 9: Test ARM emulation is working
                bat 'docker exec coinos-build bash -c "dpkg --add-architecture arm64 && apt-get update && apt-get download libc6:arm64 && dpkg -x libc6_*_arm64.deb test && ./test/lib/aarch64-linux-gnu/ld-linux-aarch64.so.1 --version || echo \\"ARM emulation test\\""'

                // List available layers
                bat 'docker exec coinos-build bash -c "cd /workspace && ./rpi-image-gen/rpi-image-gen layer --list"'

                // Step 10: Build using our config with layer search path
                bat 'docker exec coinos-build bash -c "cd /workspace && ./rpi-image-gen/rpi-image-gen build -S /workspace -c config/coinos-config.yaml"'

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