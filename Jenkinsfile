pipeline {
    agent {
        label 'arm64'
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
                    
                    # Install system dependencies (with sudo)
                    sudo apt-get update
                    sudo apt-get install -y git curl wget build-essential dosfstools
                    
                    # Verify mkdosfs is available
                    which mkdosfs || echo "mkdosfs not found, checking installation..."
                    mkdosfs --version || echo "Trying to locate mkdosfs..."
                    
                    # Sometimes mkdosfs is in /sbin which might not be in PATH for non-root users
                    if [ -f "/sbin/mkdosfs" ]; then
                        echo "Found mkdosfs in /sbin, adding to PATH"
                        export PATH="/sbin:$PATH"
                    fi
                    
                    # Clone rpi-image-gen if needed
                    if [ ! -d "rpi-image-gen" ]; then
                        git clone https://github.com/raspberrypi/rpi-image-gen.git
                    fi
                    
                    # Install rpi-image-gen dependencies (as root)
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
                    
                    # Make sure PATH includes /sbin for mkdosfs
                    export PATH="/sbin:$PATH"
                    
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
                    
                    # Look for built images - check in work directory
                    if [ -d "work" ]; then
                        echo "Checking work directory..."
                        find work -name "*.img" -o -name "*.img.*" 2>/dev/null | while read file; do
                            echo "Found: $file"
                            ls -lh "$file"
                        done || echo "No image files found in work/"
                        
                        # Also check for image-coinos-image directory
                        if [ -d "work/image-coinos-image" ]; then
                            echo "Checking image-coinos-image directory..."
                            find work/image-coinos-image -type f -name "*.img*" 2>/dev/null
                        fi
                    else
                        echo "No work directory found"
                    fi
                    
                    # Check rpi-image-gen's output directory too
                    if [ -d "rpi-image-gen/work" ]; then
                        echo "Checking rpi-image-gen/work directory..."
                        find rpi-image-gen/work -name "*.img" -o -name "*.img.*" 2>/dev/null
                    fi
                    
                    echo "=== Current directory contents ==="
                    ls -la
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                // Archive only the specific image files we want
                archiveArtifacts artifacts: 'work/deploy-*/coinos-image.img, work/deploy-*/coinos-image.img.*, build.log', fingerprint: true
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
                echo "PATH: ${PATH}"
            '''
        }
    }
}