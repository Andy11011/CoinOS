pipeline {
    agent {
        label 'arm64'  // This tells Jenkins to run on your GCP ARM64 node
    }
    
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Andy11011/CoinOS.git'
            }
        }
        
        stage('Setup Dependencies') {
            steps {
                sh '''
                    echo "=== Setting up on native ARM64 ==="
                    echo "Architecture: $(uname -m)"
                    echo "Working directory: $(pwd)"
                    
                    # Install system dependencies
                    sudo apt-get update
                    sudo apt-get install -y git curl wget
                    
                    # Clone rpi-image-gen if needed
                    if [ ! -d "rpi-image-gen" ]; then
                        git clone https://github.com/raspberrypi/rpi-image-gen.git
                    fi
                    
                    # Install rpi-image-gen dependencies
                    cd rpi-image-gen
                    sudo ./install_deps.sh
                    
                    cd ..
                '''
            }
        }
        
        stage('Build CoinOS') {
            steps {
                sh '''
                    echo "=== Building CoinOS on native ARM64 ==="
                    
                    # Build directly on ARM64 - no Docker needed!
                    # The -S flag points to source directory (current dir)
                    ./rpi-image-gen/rpi-image-gen build -S . -c config/coinos-config.yaml 2>&1 | tee build.log
                    
                    echo "=== Build completed! ==="
                '''
            }
        }
        
        stage('Check Results') {
            steps {
                sh '''
                    echo "=== Generated artifacts ==="
                    
                    # Look for built images
                    find . -name "*.img" -o -name "*.img.*" 2>/dev/null | while read file; do
                        echo "Found: $file"
                        ls -lh "$file"
                    done || echo "No image files found"
                    
                    # Check work directory
                    echo "=== Work directory contents ==="
                    ls -la work/ 2>/dev/null || echo "No work directory"
                '''
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                // Archive all image files and logs
                archiveArtifacts artifacts: '**/*.img, **/*.img.*, build.log', fingerprint: true
            }
        }
    }
    
    post {
        success {
            echo '✅ CoinOS build completed successfully on native ARM64!'
        }
        failure {
            echo '❌ CoinOS build failed!'
        }
        always {
            sh '''
                echo "=== Build completed ==="
                echo "Node: ${NODE_NAME}"
                echo "Workspace: ${WORKSPACE}"
            '''
        }
    }
}